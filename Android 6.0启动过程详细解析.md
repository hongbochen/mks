---
title: Android 6.0启动过程详细解析
tags: android, 启动过程,
grammar_cjkRuby: true
---

在之前的一篇文章中，从概念上学习了Andoird系统的启动过程，[Android系统启动过程学习][1]

  而在这篇文章中，我们将从代码角度仔细学习Android系统的启动过程，同时，学习Android启动过程中的初始化脚本语言，即`init.rc`中的语言语法。在这里，不在详细介绍Linux内核的启动过程，主要学习从Linux内核启动之后，init初始化是如何工作的，他是如何启动Android系统的第一个进程--Zygote进程。并且还会继续了解后面其他的进程是如何通过Zygote进程启动的。话不多说，我们现在就来气Android系统启动之路。
  
  ## Android系统启动流程图
  
  ![enter description here][2]


  [1]: http://blog.csdn.net/hongbochen1223/article/details/53780960
  [2]: ./images/android6.0%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90.png "android6.0系统启动流程分析.png"
  
  我们都知道，Android系统内核是基于Linux内核，所以在Android系统启动过程中，首先启动Linux内核，Bootloader加载并启动Linux内核，内核启动完成之后，内核开始启动Android系统的init进程，然后init进程通过init.rc启动脚本语言的执行，来启动Zygote进程，作为Android其他进程的父进程，Zygote进程做完初始化工作之后，启动SystemServer来启动其他系统服务。
  
 下面我们从init进程的启动开始学习。
 
 ```
  
	  int main(int argc, char** argv) {
		if (!strcmp(basename(argv[0]), "ueventd")) {
			return ueventd_main(argc, argv);
		}

		if (!strcmp(basename(argv[0]), "watchdogd")) {
			return watchdogd_main(argc, argv);
		}

		// Clear the umask.
		umask(0);

		add_environment("PATH", _PATH_DEFPATH);

		bool is_first_stage = (argc == 1) || (strcmp(argv[1], "--second-stage") != 0);

		// Get the basic filesystem setup we need put together in the initramdisk
		// on / and then we'll let the rc file figure out the rest.
		if (is_first_stage) {
			mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
			mkdir("/dev/pts", 0755);
			mkdir("/dev/socket", 0755);
			mount("devpts", "/dev/pts", "devpts", 0, NULL);
			mount("proc", "/proc", "proc", 0, NULL);
			mount("sysfs", "/sys", "sysfs", 0, NULL);
		}

		// We must have some place other than / to create the device nodes for
		// kmsg and null, otherwise we won't be able to remount / read-only
		// later on. Now that tmpfs is mounted on /dev, we can actually talk
		// to the outside world.
		open_devnull_stdio();
		klog_init();
		klog_set_level(KLOG_NOTICE_LEVEL);

		NOTICE("init%s started!\n", is_first_stage ? "" : " second stage");

		if (!is_first_stage) {
			// Indicate that booting is in progress to background fw loaders, etc.
			close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));

			property_init();

			// If arguments are passed both on the command line and in DT,
			// properties set in DT always have priority over the command-line ones.
			process_kernel_dt();
			process_kernel_cmdline();

			// Propogate the kernel variables to internal variables
			// used by init as well as the current required properties.
			export_kernel_boot_props();
		}

		// Set up SELinux, including loading the SELinux policy if we're in the kernel domain.
		selinux_initialize(is_first_stage);

		// If we're in the kernel domain, re-exec init to transition to the init domain now
		// that the SELinux policy has been loaded.
		if (is_first_stage) {
			if (restorecon("/init") == -1) {
				ERROR("restorecon failed: %s\n", strerror(errno));
				security_failure();
			}
			char* path = argv[0];
			char* args[] = { path, const_cast<char*>("--second-stage"), nullptr };
			if (execv(path, args) == -1) {
				ERROR("execv(\"%s\") failed: %s\n", path, strerror(errno));
				security_failure();
			}
		}

		// These directories were necessarily created before initial policy load
		// and therefore need their security context restored to the proper value.
		// This must happen before /dev is populated by ueventd.
		INFO("Running restorecon...\n");
		restorecon("/dev");
		restorecon("/dev/socket");
		restorecon("/dev/__properties__");
		restorecon_recursive("/sys");

		epoll_fd = epoll_create1(EPOLL_CLOEXEC);
		if (epoll_fd == -1) {
			ERROR("epoll_create1 failed: %s\n", strerror(errno));
			exit(1);
		}

		signal_handler_init();

		property_load_boot_defaults();
		start_property_service();

		init_parse_config_file("/init.rc");

		action_for_each_trigger("early-init", action_add_queue_tail);

		// Queue an action that waits for coldboot done so we know ueventd has set up all of /dev...
		queue_builtin_action(wait_for_coldboot_done_action, "wait_for_coldboot_done");
		// ... so that we can start queuing up actions that require stuff from /dev.
		queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
		queue_builtin_action(keychord_init_action, "keychord_init");
		queue_builtin_action(console_init_action, "console_init");

		// Trigger all the boot actions to get us started.
		action_for_each_trigger("init", action_add_queue_tail);

		// Repeat mix_hwrng_into_linux_rng in case /dev/hw_random or /dev/random
		// wasn't ready immediately after wait_for_coldboot_done
		queue_builtin_action(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");

		// Don't mount filesystems or start core system services in charger mode.
		char bootmode[PROP_VALUE_MAX];
		if (property_get("ro.bootmode", bootmode) > 0 && strcmp(bootmode, "charger") == 0) {
			action_for_each_trigger("charger", action_add_queue_tail);
		} else {
			action_for_each_trigger("late-init", action_add_queue_tail);
		}

		// Run all property triggers based on current state of the properties.
		queue_builtin_action(queue_property_triggers_action, "queue_property_triggers");

		while (true) {
			if (!waiting_for_exec) {
				execute_one_command();
				restart_processes();
			}

			int timeout = -1;
			if (process_needs_restart) {
				timeout = (process_needs_restart - gettime()) * 1000;
				if (timeout < 0)
					timeout = 0;
			}

			if (!action_queue_empty() || cur_action) {
				timeout = 0;
			}

			bootchart_sample(&timeout);

			epoll_event ev;
			int nr = TEMP_FAILURE_RETRY(epoll_wait(epoll_fd, &ev, 1, timeout));
			if (nr == -1) {
				ERROR("epoll_wait failed: %s\n", strerror(errno));
			} else if (nr == 1) {
				((void (*)()) ev.data.ptr)();
			}
		}

		return 0;
	}
  
  ```
  
该文件位于`system/init/init.cpp`中，我们来看看init进程都做了哪些工作。

首先，init进程添加环境变量，并且挂载相应的目录。