---
title:uboot启动过程详解
tags: uboot,启动验证
grammar_cjkRuby: true
---
在android启动过程中，首先启动的便是uboot，uboot是负责引导内核装入内存启动或者是引导recovery模式的启动。现在在很多android的uboot的启动过程中，都需要对内核镜像和ramdisk进行验证，来保证android系统的安全性，如果在uboot引导过程中，如果内核镜像或ramdisk刷入的是第三方的未经过签名认证的相关镜像，则系统无法启动，这样便保证了android系统的安全性。

在uboot启动过程中，是从`start.S`开始的，这里详细的细节不在赘述了，该篇文章主要学习uboot对内核镜像和ramdisk镜像的验证启动过程，同时学习一下里面的优秀巧妙的编码方式。

我们从`arch/arm/lib/board.c`的函数`board_init_r`函数开始，我们来看一下该代码：

```

	void board_init_r(gd_t *id, ulong dest_addr)
	{
		gd = id;

		gd->flags |= GD_FLG_RELOC;	/* tell others: relocation done */

		monitor_flash_len = _end_ofs;

		debug("monitor flash len: %08lX\n", monitor_flash_len);
		board_init();	/* Setup chipselects */

	#if defined(CONFIG_MISC_INIT_R)
		/* miscellaneous platform dependent initialisations */
		misc_init_r();
	#endif

	#if defined(CONFIG_USE_IRQ)
		/* set up exceptions */
		interrupt_init();

		/* enable exceptions */
		enable_interrupts();
		printf("init interrupt done!\n");
	#endif

	#if defined(CONFIG_COMIP_FASTBOOT) && defined(CONFIG_LCD_SUPPORT)
		if (gd->fastboot) {
		/*register lcd support*/
	#if defined (CONFIG_LCD_AUO_OTM1285A_OTP)
			extern int lcd_auo_otm1285a_otp_init(void);
			lcd_auo_otm1285a_otp_init();
	#endif
	#if defined (CONFIG_LCD_AUO_R61308OTP)
			extern int lcd_auo_r61308opt_init(void);
			lcd_auo_r61308opt_init();
	#endif
	#if defined (CONFIG_LCD_AUO_NT35521)
			extern int lcd_auo_nt35521_init(void);
			lcd_auo_nt35521_init();
	#endif
	#if defined (CONFIG_LCD_SHARP_R69431)
			extern int lcd_sharp_eR69431_init(void);
			lcd_sharp_eR69431_init();
	#endif
			/*initialize lcdc & display logo*/
			extern int comipfb_probe(void);
			comipfb_probe();
		}
	#endif

	#if defined(CONFIG_COMIP_TARGETLOADER)
		extern int targetloader_init(void);
		targetloader_init();
	#elif defined(CONFIG_COMIP_FASTBOOT)
		if (gd->fastboot) {
			extern int fastboot_init(void);
			fastboot_init();
			while(1);
		}
	#endif

	#if defined(CONFIG_ENABLE_SECURE_VERIFY_FOR_BOOT)
			extern  int secure_verify(void);
			extern void pmic_power_off(void);
			if(secure_verify()) {
					printf("Secure verify failed! Shutdown now!\n");
					pmic_power_off();
			} else {
					printf("Secure verify succeed!\n");
			}
	#endif

		do_bootm_linux();

		/* main_loop() can return to retry autoboot, if so just run it again. */
		for (;;) {
			//main_loop();
		}

		/* NOTREACHED - no way out of command loop except booting */
	}

```

首先该函数做的是初始化board，调用`board_init()`函数。该函数位于`board/****/***.c`文件中，该文件由于属于板子厂家，所以暂时保密。我们来看一下这个函数：

```

	int board_init(void)
	{
		gd->bd->bi_arch_number = MACH_TYPE_LC186X;
		gd->bd->bi_boot_params = CONFIG_BOOT_PARAMS_LOADADDR;

	#ifndef CONFIG_COMIP_TARGETLOADER
		tl420_init();

		watchdog_init();

		comip_lc186x_coresight_config();

		comip_lc186x_sysclk_config();

		comip_lc186x_sec_config();

		comip_lc186x_bus_prior_config();
	#endif

	#if defined(COMIP_LOW_POWER_MODE_ENABLE)
		comip_lp_regs_init();
	#endif
		icache_enable();
		//dcache_enable();
	#if CONFIG_COMIP_EMMC_ENHANCE
		mmc_set_dma(1);
	#endif
		flash_init();

	#ifndef CONFIG_COMIP_TARGETLOADER
		pmic_power_on_key_check();
		boot_image();
		pmic_power_on_key_check();
	#endif

	#ifdef CONFIG_PMIC_VIBRATOR
		pmic_vibrator_enable_set();
	#endif

		return 0;
	}

```

在该函数中，主要是用于初始化一些参数和硬件，包括arch版本号，boot加载地址，初始化watchdog，系统时钟，总线，flash，同时还需要做的就是，我们开机时的按钮监听，组合键按钮监听，启动镜像，开机震动等操作，在这里我们看一下`boot_image()`函数的实现。