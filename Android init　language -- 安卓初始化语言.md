## Android Init Language
## Android初始化语言

<hr />

Android初始化语言包含四个广泛的陈述类，为Actions(行为),Commands(命令),
Services(服务)和Options(选项)。

所有的这些都是基于行的，包括空格分隔符。C风格的反斜杠可以用来插入空格到一个
命令中。双引号可以被用来阻止空格将文本分割成多个标记。当反斜杠为一行中的最后
一个字符的时候，可以被用于换行。

以#开始的行为注释。

Actions和Services声明一个新的部分。所有的commands和options属于最近声明
的那个部分。位于第一个部分之前的Commands和Actions是被忽略的。

Actions和Services用于独一无二的名称。如果有一个Action或Service被声明了一个
和之前相同名称，则他被忽略为一个错误。

### 行为
<hr >

Actions是被命名的命令序列。Actions有一个触发器用来决定什么时候这个action应该发生。
当一个时间触发了符合一个action的触发器，那么这个action将被添加到要处理队列的
尾部（除非他已经在队列中了）。

在队列中的每一个action都是按照顺序出列的，位于那个action中的每一个command
也是按照顺序执行的。在活动中，初始化处理其他活动（设备创建和销毁，属性设置，
进程重启）介于commands的运行之间。

Actions组织形式位：

```

	on <trigger>
	   <command>
	   <command>
	   <command>
```

### 服务
<hr / >

服务是，当他们退出的时候，init进程启动和重新启动(可选)的程序。Services的形式为：

```

	service <name> <pathname> [ <argument> ]*
	   <option>
	   <option>
	   ...
   
```

### 选项

<hr / >

选项是服务的调节器。他们影响init进程如何并且何时运行这个服务。

	critical
	  这是一个设备关键服务。如果他在四分钟内存在超过四次，设备将会重启进入恢复模式。

	disabled
	  这个服务将不能自动起到能够他的类。他必须通过名称被显示启动。

	setenv <名称> <值>
	  在启动进程中设置环境变量<名称>到<数值>

	socket <name> <type> <perm> [ <user> [ <group> [ <seclabel> ] ] ]
	  创建一个unix域套接字，命名为 /dev/socket/<name>，并且传递他的文件描述符fd到启动进程中。
	  <type>必须是"dgram","stream"或者是"seqpacket"。
	  User和group默认为0。
	  'seclabel'是这个套接字的SELinux安全上下文。
	  他默认为服务安全上下文，由seclabel指定或者是基于服务可执行文件安全上下文计算得来。

	user <username>
	  在运行这个服务之前改为username(用户名称)。
	  当前默认为root.目前，如果你的进程需要linux功能，那么你不能使用这个命令。你必须在进程中
	  请求功能但依然root，然后降级到你期望的uid。

	group <groupname> [ <groupname> ]*
	  在运行这个服务之前修改为groupname。在第一个之前的组名称被用来设置进程的追加的组（通过setgrooups()）
	  。目前默认为root。

	seclabel <seclabel>
	  在执行这个服务之前改为seclabel。主要是从rootfs等中被services使用。
	  位于系统分区的services可以基于他们的文件安全上下文使用策略定义的转换。
	  如果未被指定或者是在策略中没有转换被定义，默认为初始化上下文。

	oneshot
	  当他退出的时候不要重启服务。

	class <name>
	  为服务指定一个类名称。位于一个命名的类中的所有服务一起被启动或停止。
	  如果服务未被class选项指定，则该服务位于类'default'中。

	onrestart
	  当services重启的时候运行一个命令。

	writepid <file...>
	  当子进程被创建的时候，将子进程的pid写入到给定的文件中。意味着cgroup/cpuset
	  用法。

### 触发器
<hr />

触发器是一系列字符串，被用于匹配确定种类的事件，并且被用于出发一个行为。

	boot
	   这是第一个触发器，当init进程启动的时候被触发（在/init.conf被加载之后）。

	<name>=<value>
	   当属性<name>被设置为指定的值<value>的时候，这种形式的触发器被触发。
	   这个可以测试多个属性去运行一组命令。例如：

	   on property:test.a=1 && property:test.b=1
		   setprop test.c 1

	   上面的一小段只有当test.a=1和test.b=1都被设置的时候，才会设置test.1为1。

### 命令
<hr />

	bootchart_init
	   如果被配置的话，开启bootcharting。他被默认包含在init.rc中。

	chmod <octal-mode> <path>
	   修改文件访问权限。

	chown <owner> <group> <path>
	   修改文件拥有者和组。

	class_start <serviceclass>
	   如果他们没有运行的话，启动所有指定类的服务。

	class_stop <serviceclas
	   如果他们当前正在运行的话，停止并且禁用所有指定类的服务。

	class_reset <serviceclass>
	   如果他们正在运行的话，停止所有指定类的服务，并不禁用他们。后期他们使用
	   class_start被重新启动。

	copy <src> <dst>
	   复制一个文件。类似于写，但是对二进制/大数量数据是有用的。

	domainname <name>
	   设置域名称。

	enable <servicename>
	   将一个禁用的服务转换为可用，好像这个服务并没有被指定禁用过。
	   如果服务期望去运行，他将会现在被启动。特别是当bootloader设置了一个变量来指示一个指定
	   的服务应该在被需要的时候启动。
		  on property:ro.boot.myfancyhardware=1
			enable my_fancy_service_for_my_fancy_hardware

	exec [ <seclabel> [ <user> [ <group> ]* ] ] -- <command> [ <argument> ]*
	   使用给定的参数创建并且执行命令。这个命令在"--"之后运行，所以一个可选的安全上下文，用户，
	   和追加的组可以被提供。在这个结束之前没有其他的命令可以被运行。<seclabel>可以被一个-来
	   指定为默认。

	export <name> <value> 
	   在全局环境中设置环境变量<name>等于<value>。（这个命令执行之后启动的所有
	   进程都将继承他。）

	hostname <name>
	   设置主机名

	ifup <interface>
	   将网络接口<interface>联机。

	import <filename>
	   解析一个初始化配置文件，扩展当前的配置。

	insmod <path>
	   在<path>中安装模块

	load_all_props
	   从/system,/vendor等中加载属性。这个被包含在默认的init.rc中。

	load_persist_props
	   当/data被加密之后，加载持久化的属性。
	   这个被包含在默认的init.rc中。

	loglevel <level>
	   一层一层的设置内核日志。属性被扩展到<level>中。

	mkdir <path> [mode] [owner] [group]
	   在路径<path>中创建一个目录，可选的使用给定的模式，所有者和组。如果没有提供，该目录则
	   以权限755被创建并且以root用户为拥有者和以root为组。如果提供了，模式，所有者和组将会被
	   更新，如果该目录已经存在的话。

	mount_all <fstab>
	   在给定的fs_mgr-format fstab中调用fs_mgr_mount_all。

	mount <type> <device> <dir> [ <flag> ]* [<options>]
	   尝试在目录<dir>中挂载命名的设备。<device>可以是mtd@name的形式来通过名称
	   指定一个mtd块设备。
	   <flag>s包含"ro","rw","remount","noatime"等。
	   <options>包含"barrier=1", "noauto_da_alloc", "discard"等作为一个逗号分离的字符串。例如，
		barrier=1,noauto_da_alloc

	powerctl
	   内部实现系统用来响应改变"sys.powerctl"属性，用于实现重启。

	restart <service>
	   类似于stop，但是并不禁用服务。

	restorecon <path> [ <path> ]*
	   重新以file_contexts配置中指定的安全上下文来将文件重新以<path>存储。
	   不需要被init.rc创建的目录，因为这些都是被init初始化进程自动正确标记的。

	restorecon_recursive <path> [ <path> ]*
	   递归的重新以名称<path>将目录树保存到安全上下文中，这个安全上下文由
	   file_contexts配置指定。

	rm <path>
	   在指定的路径调用unlink(2)。你可能想要使用"exec -- rm ..."（提供的系统
	   分区已经被挂载了）。

	rmdir <path>
	   对给定的目录调用rmdir(2)

	setprop <name> <value>
	   这是系统属性<name>到<value>。属性被扩展到<value>。

	setrlimit <resource> <cur> <max>
	   为一个资源设置rlimit。

	start <service>
	   开启一个服务运行，如果这个服务没有运行的话。

	stop <service>
	   从运行的服务中停止一个服务，如果这个服务正在运行的话。

	swapon_all <fstab>
	   对给定的fstab文件执行fs_mgr_swapon_all

	symlink <target> <path>
	   使用值<target>在<path>中创建一个符号链接。

	sysclktz <mins_west_of_gmt>
	   设置系统时钟基础（如果系统时钟在格林尼治时间为0）

	trigger <event>
		触发一个事件。用于从另一个动作中排队一个动作。

	verity_load_state
	   内部实现细节用于加载dm-verity状态。

	verity_update_state <mount_point>
	  内部实现细节用于更新dm-verity状态并且设置分区。<mount_point>被adb remount
	   使用来验证属性，因为fm_mgr不能直接自己设置他们。

	wait <path> [ <timeout> ]
	   轮询给定文件的存在性，并且当找到的时候返回，或者是到达超时状态。
	   如果超时未被指定，则默认为5秒。

	write <path> <content>
	   在路径<path>中打开文件，并且使用write(2)写一个字符串进去。
	   如果文件不存在，他将被创建。如果文件存在，他将被覆盖。属性在<content>
		被扩展。

### 属性

<hr />

init进程更新一些系统属性，并且提供一些他要干什么的信息：

	init.action
	   相当于当前被执行的action的名称，如果不存在则为""。

	init.command
	   相当于当前被运行的command，如果没有则为""。

	init.svc.<name>
	   一个被命名的服务的状态("停止","运行","重启")。

### Bootcharting

<hr />

init的这个版本包含运行"bootcharting"的代码：生成一个日志文件，后期能够被 www.bootchart.org
提供的工具处理。

在虚拟机中，使用-bootchart <timeout>选项来使启动的时候带有bootcharting持续<timeout>秒。

在一个设备中，使用命令创建 /data/bootchart/start：
　　`adb shell 'echo $TIMEOUT > /data/bootchart/start'`

$TIMEOUT的值对应着期望bootchart持续的秒数。当这些时间过后，Bootcharting将会停止。

你可以通过下面的命令在任何时候停bootcharting：
　　`adb shell 'echo 1 > /data/bootchart/stop'`

注意，/data/bootchart/stop会在bootcharting最后被init自动删除。对于/data/bootchart/start
情况并非如此，所以当你收集完数据之后，不要忘记删除他。

日志文件被写入/data/bootchart中。一个脚本被提供去恢复他们，并且创建一个bootchart.tgz文件
，这个文件可以被bootchart命令行工具使用：

```

	  sudo apt-get install pybootchartgui
	  # grab-bootchart.sh uses $ANDROID_SERIAL.
	  $ANDROID_BUILD_TOP/system/core/init/grab-bootchart.sh

```

一个需要注意的事情就是，bootchart将会显示init好像是他从0s的时候开始运行。当内核
开始init的时候，你必须查看dmesg的工作。

### Debugging init

<hr / >

默认的，由init执行的程序将会把标准输出和标准错误丢入到/dev/null。为了帮助调试，
你可以通过安卓程序日志封装程序运行你的程序。这个将会重定向标准输出/标准错误到
安卓日志系统中。

例如
`service akmd /system/bin/logwrapper /sbin/akmd`

当在init中自己运行的时候，为了快速的转变，使用：

```

	  mm -j
	  m ramdisk-nodeps
	  m bootimage-nodeps
	  adb reboot bootloader
	  fastboot boot $ANDROID_PRODUCT_OUT/boot.img

```

可选的，使用虚拟机：

 ` emulator -partition-size 1024 -verbose -show-kernel -no-window`

在klog_init()调用之后，你可能想要调用klog_set_level(6)，所以你需要在
dmesg或者是虚拟机输出中查看内核日志。



