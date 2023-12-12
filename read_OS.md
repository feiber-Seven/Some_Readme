没有懂的：
	1.GDT（全局内存表）--> 
		a.段描述符：唯一对应一个段选择子，描述（不同的段，如代码段、数据段、堆栈段等）
		b.段选择子：索引（段描述符在G/L表中的位置）+ 标志位（是GDT还是LDT）
		c.在使用选择子前，加载GDTR/LDTR（寄存器），找到段描述符，如何得到段选择子，加载到对应段寄存器（CS、DS、SS等）
		d.段选择子和段寄存器 组合-->cpu确定内存具体位置、段描述符-->访问权限和保护检查
	(ok)2.p467 为什么输出结果应该是 32MB
	3.p547 刷新改良
	4.p600 map
	(ok)5.p649 time优化
	6.int20 阅读
	7.p925 锁定键led控制
	8.p1160 cpu优化执行异常int
	9.p1263 linewin()

CALL 和 JMP : call会跳回，jmp接着执行下去

一个汇编程序变成API的过程: (我希望能输出一个字符)
	前提: 我有一个C语言函数void cons_putchar(struct CONSOLE *cons, int chr, char move)，并且我将hello.nas注册到GDT的1003号
	1.将待显示的字符 'A'->AL寄存器 调用cons_putchar(x,x,x)显示除来
		need.执行CALL: 需要知道cons_putchar(x,x,x)函数的地址
		question.函数cons_putchar(x,x,x)是C语言编写，接收堆栈里面的数据，接收不到AL寄存器
	2.我们便需要一个(不在应用程序部分，而是在操作系统naskfunc.nas部分的)_asm_cons_putchar: PUSH EAX - PUSH (cons的地址)，让hello.nas程序CALL这个_asmxx函数
		need.找到cons的地址: 还未定义，不妨放在0x0fec(添加到console_task里: 定义*((int *) 0x0fec) = (int) &cons;)
		question.目前不知道_asm函数的地址
	3.make找到haribote.map，可以找到类似:"0x00000BE3 : _asm_cons_putchar"，API命令需要使用Far-CALL，所以命令为: CALL 2*8:0xbe3。并在被F-C指令后加入'RETF'
	4.我需要在C语言里面写函数调用hello.nas的函数
		need.C语言里面调用farcall()
	5.在naskfunc.nas里面加入_far_call供使用，C语言: farcall(0, 1003*8)。由于我们修改了操作系统汇编代码，_asm_cons_putchar在map里的位置会发生改变，要在hello.nas里面修该Far-CALL地址
		need.静态化_asmxx地址
	6.在IDT里面注册函数，不妨设为0x40号，所以在init_gdtidt()里面，需要set_gatedesc(idt + 0x40, (int)函数名:_asmxx, 2 * 8, AR_INTGATE32);此时，hello.nas通过F-C调用_asmxx是INT方式，_asmxx需要IRETD返回。
	*总结: 
	C语言(运用在汇编实现的far_call()) 
	---> 通过GDT调用hello.nas 
	---> 通过INT/IDT调用nask.nas里面的_asmxx函数 
	---> 通过PUSH 1/ PUSH EAX/ PUSH DWORD [0x0fec]/ ADD ESP,12/ 将参数传入C语言函数

APP(文件形式存储代码)实现：
	主main() 
	--(调用console.c)-> 打开console 
	--(输入命令)-> 识别APP 
	--(找寻app.c文件)-> 文件main函数调用功能|例如:api_point() 
	--(堆栈传值)--> 在a_nask.nas找到汇编函数_api_point() 
	--(保存寄存器、堆栈取值、INT40)-> 找到naskfunc.nas里_asm_hrb_api() 
	--(CALL(console.c里面的函数)hrb_api、保存寄存器、堆栈传全寄存器)-> hrb_api()接收寄存器、调用相应实现xx功能的函数|例:控制一个像素点

内存分布：
	0x00000000 - 0x000fffff : 虽然在启动中会多次使用，但之后就变空。（1MB）
	0x00100000 - 0x00267fff : 用于保存软盘的内容。（1440KB）
	0x00268000 - 0x0026f7ff : 空（30KB）
	0x0026f800 - 0x0026ffff : IDT （2KB）
	0x00270000 - 0x0027ffff : GDT （64KB）
	0x00280000 - 0x002fffff : bootpack.hrb（512KB）
	0x00300000 - 0x003fffff : 栈及其他（1MB）
	0x00400000 - laaaaaaast : 空

寄存器分布：
	backlink: 指向前一个TSS的指针，用于实现任务切换时的链表结构。
	esp0: 内核栈的栈顶指针，用于任务切换时从用户态切换到内核态。---->分配内存空间，*((int *) (task_cons->tss.esp + 4)) = (int) sht_cons; //将CMD图层地址放到CMD任务块首部
	ss0: 内核数据段选择子，用于任务切换时从用户态切换到内核态。
	esp1: 暂时未使用。
	ss1: 暂时未使用。
	esp2: 暂时未使用。
	ss2: 暂时未使用。
	cr3: 页目录基址寄存器（Page Directory Base Register，PDBR），存储了任务的页表信息。
	eip: 任务的指令指针，记录了任务下一条要执行的指令地址。 ---->选择要执行的程序段，task->tss.eip指向这个任务要运行的操作
	eflags: 任务的标志寄存器，存储了各种标志位，如进位标志、零标志等。
	eax, ecx, edx, ebx, esp, ebp, esi, edi: 通用寄存器，用于存储任务的通用数据。
	es, cs, ss, ds, fs, gs: 任务的段选择子，用于指定任务在不同段寄存器中的段描述符。
	ldtr: 局部描述符表寄存器（Local Descriptor Table Register），存储了LDT的段选择子。
	iomap: I/O位图基址，用于限制任务对I/O端口的访问权限。

文件记录：
	struct FILEINFO {
		unsigned char name[8]; 
			1.前8个字节是文件名，不足8个字节时，用空格补足。 
			2.所有的文件名都是大写的 
			3.第一个字节0xe5（文件已被删除）0x00（不包含任何文件名信息）
		unsigned char ext[3]; 
			1.类比文件名的3个字节：扩展名
		unsigned char type; 
			1.一个字节的属性信息
			2.每一位有特殊含义
				0x01：只读文件（不可写入）
				0x02：隐藏文件
				0x04：系统文件
				0x08：非文件信息（比如磁盘名称等）
				0x10：目录
		char reserve[10];
			1.10字节保留
		unsigned short time, date;
			1.2字节time 和 2字节date
		unsigned short clustno;
			1.2字节：文件内容从磁盘上的哪个扇区开始存放
			2.clustno：“簇号”（cluster number）
			3.磁盘映像中的地址 = clustno * 512 + 0x003e00
		unsigned int size;
			1.4字节存放文件大小
	};


ipl10.nas 是 启动boot汇编文件，对磁盘加载基础内容 起始在0x7c00处


asmhead.nas 是 包含了：
			1.图形分辨率设置 （是否VBE）
			2.获取键盘led状态（INT16） 
			3.禁止所有中断（通过CLI控制pic）
			4.启动A20地址线（为了cpu访问1mb以上的内存空间）
			**5.设置GDT，加载GDTR（cr0-->系统模式）
			*6.将磁盘数据和bootpack（.hrb）复制到内存（memcoy([esi]源,[edi],[ecx])）
			7.waitkbdout
			8.启动bootpack
            的开机引导汇编程序，起始在0xc200
               重点在：


naskfunc.nas 是 用于解决C语言无法解决的联系硬件问题，定义部分C语言所需要的函数，包含了：
	_io_hlt: 执行 HLT 指令，使 CPU 进入停机状态。
	_io_cli: 执行 CLI 指令，禁止外部中断。
	_io_sti: 执行 STI 指令，允许外部中断。
	_io_stihlt: 执行 STI 指令，然后执行 HLT 指令，允许外部中断并使 CPU 进入停机状态。
	_io_in8: 从指定的 I/O 端口读取一个字节的数据。
	_io_in16: 从指定的 I/O 端口读取一个字（16 位）的数据。
	_io_in32: 从指定的 I/O 端口读取一个双字（32 位）的数据。
	_io_out8: 将一个字节的数据写入指定的 I/O 端口。
	_io_out16: 将一个字（16 位）的数据写入指定的 I/O 端口。
	_io_out32: 将一个双字（32 位）的数据写入指定的 I/O 端口。
	_io_load_eflags: 获取 EFLAGS 寄存器的值。
	_io_store_eflags: 将指定的值存储到 EFLAGS 寄存器。
	_load_gdtr: 加载全局描述符表（GDT）的限制和基地址。
	_load_idtr: 加载中断描述符表（IDT）的限制和基地址。
	_load_tr: 
	_asm_inthandler21: 中断处理程序，处理 IRQ 0x21 中断。
	_asm_inthandler27: 中断处理程序，处理 IRQ 0x27 中断。
	_asm_inthandler2c: 中断处理程序，处理 IRQ 0x2C 中断。
	_farjmp: 多任务跳转程序，执行JMP FAR。


C语言文件中：
	1.bootpack.h 头文件
		1./* asmhead.nas */
			struct BOOTINFO(屏幕相关参数):
					a.cyls-柱面（磁头）数目
					b.leds-键盘led
						{
							MOV BX,0x4101 ;VBE的640x480x8bi彩色:103,105,107分别有其他的分辨率
							MOV AX,0x4f02 ;1.采用VBE模式：AX = 0x4f02；BX = 画面模式号码
										   2.不采用VBE模式：AH =0；AL=画面模式号码
							INT 0x10
						}
					c.vmode-显示模式（文本、图形）
					d.reserve-保留
					e.scrnx scrny-屏幕的分辨率（宽度和高度）像素数量
					f.*vram-屏幕显示内存的起始地址first

		2./* naskfunc.nas */ 《保存函数》
			void io_hlt(void); 停机
			void io_cli(void); 禁止中断
			void io_sti(void); 允许中断
			void io_stihlt(void); STI && HLT
			int io_in8(int port); 从port（端口地址）读取一个8位数据
			void io_out8(int port, int data); 将data写到port
			int io_load_eflags(void); 获取eflags（32位：CF进位、PF奇偶、AF辅助进位、ZF零、SF符号、OF溢出）
			void io_store_eflags(int eflags); 存储一个值到eflags
			void load_gdtr(int limit, int addr); addr为基址limit为界限，加载GDT寄存器（GDTR）
			void load_idtr(int limit, int addr); 同，加载IDT寄存器
			int load_cr0(void); 读取CR0的值
			void store_cr0(int cr0); 将cr0的值存入CR0寄存器
			void load_tr(int tr); cpu记住当前运行的任务，“taskregister”（任务寄存器）此例存放在(gdt+n) -> tr = n * 8;
			void asm_inthandler20(void); 计时器中断
			void asm_inthandler21(void); 保留中断
			void asm_inthandler27(void); 键盘中断
			void asm_inthandler2c(void); 鼠标中断
			unsigned int memtest_sub(unsigned int start, unsigned int end); 测试内存，从start到end
			void farjmp(int eip, int cs); [ESP+4]这个位置就存放了eip的值，[ESP+8]存放了cs的值
			void farcall(int eip, int cs); 和farjmp类似
			void asm_hrb_api(void); 
			void start_app(int eip, int cs, int esp, int ds, int *tss_esp0);
			void asm_end_app(void); 结束app回到最开始读取app命令时

		3./* fifo.c */: first:mousefifo, keyfifo, timerfifo / second: fifo
			struct FIFO8(字节型缓冲区): #还有一个FIFO32是类似的，只有char类型和int类型的区别
					a.*buf-分配缓冲区的起始地址
					b.p-下一个可写入数据的索引
					c.q-下一个可读取数据的索引
					d.size-缓冲区总大小
					e.free-缓冲区剩余空间大小
					f.flags-标志位、状态信息

			void fifo32_init(struct FIFO8 *fifo, int size, unsigned char *buf); 初始化循环缓冲区，给出需要的size和*buf，建立一个*fifo
			int fifo32_put(struct FIFO8 *fifo, unsigned char data); 写入一个data，在（*buf+p？），判断full。如果这个任务此时休眠，唤醒他。
			int fifo8_get(struct FIFO8 *fifo); 读取返回 data or -1
			int fifo8_status(struct FIFO8 *fifo); 检查读取free

		4./* graphic.c */ 《屏幕绘制》
			#define定义的各种颜色与对应值
			void init_palette(void); init调色板，初始化一组颜色
			void set_palette(int start, int end, unsigned char *rgb); set调色板，其中（0x03c8是调色板的索引寄存器，0x03c9是调色板的数据寄存器），连续三个char代表一个颜色
			void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1); 在矩形（x0,x1,y1,y0）里面填充颜色c，vram-存放像素的缓冲区（屏幕首地址），xsize-行宽度
			void init_screen8(char *vram, int x, int y); init屏幕，变为整块颜色
			void putfont8(char *vram, int xsize, int x, int y, char c, char *font); 在（x，y）处绘制字符*font，以颜色c
			void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s); 调用putfont8（）实现绘制字符串
			void init_mouse_cursor8(char *mouse, char bc); *mouse保存鼠标色块，背景颜色bc，绘制鼠标
			void putblock8_8(char *vram, int vxsize, int pxsize,int pysize, int px0, int py0, char *buf, int bxsize); 绘制一个宽pxsize高pysize的图像块；用*buf存放数据，长度为bxsize

		5./* dsctbl.c */ 《中断表和文件管理表》
			struct SEGMENT_DESCRIPTOR(段描述符):
					a.short limit_low, base_low;
					b.char base_mid, access_right;
					c.char limit_high, base_high;
			struct GATE_DESCRIPTOR(门描述符):
					a.short offset_low, selector;
					b.char dw_count, access_right;
					c.short offset_high;

			void init_gdtidt(void); init gdt和idt | 可以(添加INT)操作
			void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar);
			void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar);

		6./* int.c */ 《中断》
			#define定义了PIC_xx各种操作的
			void init_pic(void); init-pic（均ICW1-4） pic0（IRQ0--7、INT20--27）和pic1（连接在pic0的IRQ2上，IRQ8--15、INT28--2f）
			void inthandler27(int *esp); INT27由电气噪音引起，不需要操作，只是通知

		7./* keyboard.c */ 《键盘》
		#define定义了KEY（DTA|CMD）
			void inthandler21(int *esp); INT21
			void wait_KBC_sendready(void);for(;;)无限循环 检查是否准备好接收数据，KEYSTA_SEND_NOTREADY == 0 表示接收
			void init_keyboard(void); init键盘
			extern strtuct FIFO8 keyfifo; 定义一个全局keyfifo缓冲区

		8./* mouse.c */ 《鼠标》
			struct MOUSE_DEC(鼠标状态信息):
					a.buf[3]-char存入鼠标数据字节
					b.phase-解码状态
					c.x y-水平、垂直状态
					d.btn-鼠标 按钮状态

			void inthandler2c(int *esp); INT2c
			void enable_mouse(struct MOUSE_DEC *mdec); 启用鼠标，接收*mdec，跟踪鼠标信息（phase），存储在*mdec中返回
			int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat); 解码*mdec中的数据，并更新*mdec；dat的来源是mousefifo缓冲区里面的值，处理（& |）完毕后存入*buf
			extern struct FIFO8 mousefifo; 定义全局mousefifo缓冲区

		9./* memory.c */ 《内存》
			struct FREEINFO(记录内存):
					a.addr-起始地址
					b.size-内存块大小
			struct MEMMAN(内存管理器):
					a.free-当前可用内存块数量
					b.maxfrees-可用最大内存块数量
					c.lostsize-内存不足丢弃的内存块总大小
					d.losts-丢弃的内存块数量
					e.free[]-freeinfo结构体数组

			unsigned int memtest(unsigned int start, unsigned int end); test测试内存再范围（start，end），监测版本386or486，修改cr0
			void memman_init(struct MEMMAN *man); init-mamman
			unsigned int memman_total(struct MEMMAN *man); memman指向当前可用总内存
			unsigned int memman_alloc(struct MEMMAN *man, unsigned int size); memman里分配一段size大小的内存
			int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size); 从memman释放addr处大小size的内存（考虑内存合并）
			unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size); memman_alloc x 4k :（size = (size + 0xfff) & 0xfffff000）
			int memman_free_4k(struct MEMMAN *man, unsigned int addr, unsigned int size); memman_free x 4k

		10./* sheet.c */ 《图层控制》
			struct SHEET(一个图层/窗口):
					a.*buf-图层像素缓冲区
					b.bxsize-宽度
					c.bysize-高度
					d.vx0 vy0-(vx0, vy0)是图层在屏幕上的坐标
					e.col_inv-透明色
					f.height-图层的高度（叠加）
					g.flags-图层标志位
					h.*ctl-指向struct SHTCTL图层控制位（有了这个，函数传入参数不需要传入SHTCTL）
					i.*task-(struct TASK)表明任务是不是将要结束的app任务: app结束task时要关闭sheet图层。sht->task = 0;不自动关闭
			struct SHTCTL(图层控制器):
					a.*vram-像素内存（屏幕首地址）
					b.*map-图层上像素叠加状态
					c.xsize-宽度
					d.ysize-高度
					e.top-当前图层最高高度
					f.*sheets[]-每个元素都是指针，指向一个图层（sheets0里面的一项），按照顺序排列
					g.sheets0[]-用于存入sheet图层，保存各项数据

			struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize); init-shtctl
			struct SHEET *sheet_alloc(struct SHTCTL *ctl); 创建一个图层sheet
			void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv); 设立图层sht缓冲区buf，宽xsize，长ysize，透明色colinv
			void sheet_updown(struct SHEET *sht, int height); 调整sht->ctl->sheet[]->height = height
			void sheet_refresh(struct SHTCTL *ctl); 刷新页面（最常变化修改优化）
			    ***a.sheet_refreshmap（刷新map）
				b.sheet_refreshsub（刷新sub）
			void sheet_slide(struct SHEET *sht, int vx0, int vy0); 滑动窗口sht到（vx0，vy0）
			void sheet_free(struct SHEET *sht); 释放sht，不使用sht

		11./* timer.c */ 《计时器》
			struct TIMER
					e.*next-next_timer指针，指向下一个TIMER串成一个链表
					a.timeout-超时时间（与timerctl.count比较超时）
					b.flags-计时器是否使用
					f.flags2-是否是应用程序计时器，是否需要取消
					c.fifo-超时时将data存进fifo
					d.data-记录超时
			struct TIMERCTL
					a.count-计数每秒100次（中断发生次数、系统运行时间）
					b.next-next_timeout下一个会timeout的TIMER的时间
					c.using-正在使用的计时器数量
					(delete)*d.*timers[]-存储所有的定时器（STRUCT TIMER），每个项指向一个定时器，按顺序排列(Timer里面存在*next，不需要*timers[]);取而代之是 *t0 --> *head
					(new)*d.*t0-*head
					e.timers0[]-存储实际定时器数据（STRUCT TIMER）

			void init_pit(void); 初始化pit（总是有一个哨兵在最后，避免完全超时）
			struct TIMER *timer_alloc(void); 加入一个timer，标注为可使用
			void timer_free(struct TIMER *timer); flag = 0，释放
			void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data); 初始化timer和有关的*fifo和data
			void timer_settime(struct TIMER *timer, unsigned int timeout); 设置超时时间，用链表排序（绝对超时时间大小），插入到链表中, timeout-->0.01s单位
			void inthandler20(int *esp); int20、IRQ0
			int timer_cancel(struct TIMER *timer); 取消指定定时器
			void timer_cancelall(struct FIFO32 *fifo); 总体 取消/释放 定时器
			
			extern struct TIMERCTL timerctl;

		12./* mtask.c */ 《多任务》
			struct TSS32
					见上“寄存器”
			struct TASK 类似tcb
					a.sel-存放GDT编号（选择子）
					b.flags-表明任务有没有运行
					c.level-任务等级
					d.priority-任务优先级
					e.fifo-缓冲区
					f.tss-(struct TSS32)
					g.*cons-(struct CONSOLE)指定是在cons[x]哪一个终端
					h.ds_base-应用数据段地址
			struct TASKLEVEL
					a.running-此Level拥有的Task数量
					b.now-正在运行的Task
					c.*tasks-(struct TASK)此Level的所有任务
			struct TASKCTL
					a.now_lv-现在的Level
					b.lv_change-下一次切换任务时需不需要切换Level
					c.level[]-(struck TASKLEVEL)存在的level的列表
					d.tasks0-(struck TASK)存在的Task的列表

			//内部
			void task_add(struct TASK *task); 增加*task到Level
			void task_remove(struct TASK *task); 移除*task，判断位置+设置task为睡眠
			void task_switchsub(void); taskctl->now_lv找到优先级最高的level(0--10)，然后切换任务完毕
			void task_idle(void); 哨兵Task
			//对外接口
			struct TASK *task_now(void); 取出Level->tasks[Level->now]
			struct TASK *task_init(struct MEMMAN *memman); init任务Task，并创建一个最低Level的哨兵
			struct TASK *task_alloc(void); 创建一个Task，初始化TSS
			void task_run(struct TASK *task, int level, int priority); 指定下一个运行的*task，并将他装到level里，并指定priority
			void task_switch(void); 切换任务
			void task_sleep(struct TASK *task); 睡眠（自我睡眠（切换任务）、他人睡眠）
			extern struct TIMER *mt_timer;

		13./* window.c */ 《窗口》
			void make_window8(unsigned char *buf, int xsize, int ysize, char *title, char act); 创建一个弹窗，图像数据存储在由buf指向的内存中，宽度xsize，高度ysize，标题title，窗口的各个元素的外观
			void putfonts8_asc_sht(struct SHEET *sht, int x, int y, int c, int b, char *s, int l); 包含 boxfill8（填充矩形颜色）、putfonts8_asc（绘制字符串）、sheet_refresh（刷新图层页面） 三个函数
			void make_textbox8(struct SHEET *sht, int x0, int y0, int sx, int sy, int c); 创建一个文本框，在*sht图层里面，起点在（x0，y0），以宽sx，长sy，背景颜色c创建
			void make_wtitle8(unsigned char *buf, int xsize, char *title, char act); 绘制窗口的标题栏内容，如标题文本和活动窗口标志。act控制是否不被选中变灰
			
		14./* console.c */ 《终端》
			void console_task(struct SHEET *sheet, unsigned int memtotal); 创建的终端-多任务，包含命令行命令（mem、clr、dir、type），sheet控制窗口，memtotal总内存
			int cons_newline(int cursor_y, struct SHEET *sheet); 刷新新的一行，伴随换行和滚动，输入行数cursor_y，返回新的行数cursor_y
			void cons_putchar(struct CONSOLE *cons, int chr, char move); 1.先判断是否为（制表符、回车符、换行符） 2.普通的字符：调用putfonts8_asc_sht函数在光标位置输出该字符。move ！= 0 移动光标8个像素。 
			void cons_putstr0(struct CONSOLE *cons, char *s); 接收到0结束的字符串输入法
			void cons_putstr1(struct CONSOLE *cons, char *s, int l); 告诉长度为l的字符串输入法
			void cons_runcmd(char *cmdline, struct CONSOLE *cons, int *fat, unsigned int memtotal); 包含（men、dir、cls、type、）系统cmd命令的调用
			void cmd_mem(struct CONSOLE *cons, unsigned int memtotal); 拆分出来的mem命令
			void cmd_cls(struct CONSOLE *cons); cls命令
			void cmd_dir(struct CONSOLE *cons); dir命令
			void cmd_type(struct CONSOLE *cons, int *fat, char *cmdline); type命令
			int cmd_app(struct CONSOLE *cons, int *fat, char *cmdline); 除去系统cmd命令以外的APP/API命令，在fat里面寻找命令文件。
				(定义代码段、数据段地址)
			void hrb_api(int edi, int esi, int ebp, int esp, int ebx, int edx, int ecx, int eax); 执行api的值，传入寄存器的值，判断EDX的值判断做出
				(1.char\
				2.string0\
				3.stringl\
				4.end\
				5.openwin\
				6.putstrwin\
				7.boxfilwin\
				8.
				9.
				10.
				11.
				12.
				13.
				14.
				15.
				16.
				17.
				18.
				19.)哪一个函数调用
			int *inthandler0d(int *esp); 
			int *inthandler0c(int *esp);
			void hrb_api_linewin(struct SHEET *sht, int x0, int y0, int x1, int y1, int col);

		15./* file.c */ 《文件管理和文件描述表FAT的关系》
			struct FILEINFO
					见上文件记录
			void file_readfat(int *fat, unsigned char *img); 解压缩文件分析表（FAT）存储到数组fat中
			void file_loadfile(int clustno, int size, char *buf, int *fat, char *img); 在文件系统中加载文件内容 到缓冲区中。找到clustno扇区，文件size大小。buf：指向存储文件内容的缓冲区的指针。fat：文件分配表（File Allocation Table）的指针。img：表示整个文件系统映像的字符数组的指针。
			struct FILEINFO *file_search(char *name, struct FILEINFO *finfo, int max);

	2.dsctbl.c gdt（分段）、idt（中断）
	3.graphic.c 屏幕显示：界面、鼠标的标志
	4.int.c 中断（!21，27，!2c）pic0、pic1
	5.keyboard.c 键盘（21int）
	6.mouse.c 鼠标（2cint）
	7.memory.c 内存（menman内存管理）   
	8.sheet.c 图层叠放
	9.timer.c 定时器
	10.mtask.c 多任务

	last_but_important.bootpack.c 运行总结项
		
		修改成cons[0] cons[1]
		HariMain()-->fifo_buf
			1.256~511确定是键盘数据
				1.256~256+0x80是键盘的扫描码（有用的按键），其他的设为0
				2.A~Z，a~z，通过 key_leds 和 key_shift 变换
				3.考虑 key_to(判断在哪个窗口) 读取s[0]
				4.256 + 0x0e 退格
				5.256 + 0x1c 回车
					考虑是否在命令行执行命令
				6.256 + 0x0f tab-切换窗口
				7.256 + 0x2a 左shift按下，key_shift第一位置为1---256 + 0xaa 左shift释放，key_shift第一位为置为0
				8.256 + 0x36 右shift按下，key_shift第二位置为1---256 + 0xb6 右shift释放，key_shift第二位为置为0
				9.256 + 0x3a CapsLock
				10.256 + 0x45 NumLock
				11.256 + 0x46 ScrollLock
				12.256 + 0xfa 键盘确认接收数据，key_wait置为-1
				13.256 + 0xfe 接收失败，重新发送数据
				14.
		1.内存检查处理（反转检查last）
		2.构建
		3.for检查中断（鼠标-（移动，拖动win窗口）
					、键盘-（显示a码（转换成大小字母/符号/数字），读取keytable[]-显示ABC，shift，Lock&Led，回车）
					、定时器-（超时，光标（用cursor_c=-1表示光标灭））
					、多任务切换-（TSS编写，TCB，多任务触发键盘）
					、窗口滚动（遍历每个像素块等于下面一行的像素块）
					、（在console_task命令行里面）mem，dir，cls指令）

		void keywin_off(struct SHEET *key_win); 关闭指定窗口的标题，并根据窗口的flags属性决定是否向与该窗口关联的任务的FIFO队列中放入值3，以控制控制台的光标关闭
		void keywin_on(struct SHEET *key_win); 打开指定窗口


app文件: 
	1./* a.nas */ 《为hrb_api()准备的api函数接口》7
		_api_putchar:	; void api_putchar(int c);
		_api_putstr0:	; void api_putstr0(char *s);
		_api_end:	; void api_end(void);
		_api_openwin:	; int api_openwin(char *buf, int xsiz, int ysiz, int col_inv, char *title);
		_api_putstrwin:	; void api_putstrwin(int win, int x, int y, int col, int len, char *str);
		_api_boxfilwin:	; void api_boxfilwin(int win, int x0, int y0, int x1, int y1, int col);
		_api_initmalloc:	; void api_initmalloc(void);
		_api_malloc:		; char *api_malloc(int size);
		_api_free:			; void api_free(char *addr, int size);
		_api_point:		; void api_point(int win, int x, int y, int col);
		_api_refreshwin:	; void api_refreshwin(int win, int x0, int y0, int x1, int y1);
		_api_linewin:		; void api_linewin(int win, int x0, int y0, int x1, int y1, int col);
		_api_closewin:		; void api_closewin(int win);
		_api_getkey:		; int api_getkey(int mode);
		_api_alloctimer:	; int api_alloctimer(void);
		_api_inittimer:		; void api_inittimer(int timer, int data);
		_api_settimer:		; void api_settimer(int timer, int time);
		_api_freetimer:		; void api_freetimer(int timer);
		_api_beep:			; void api_beep(int tone);

	2./* noolde */
	3./* beep */
		beepup
		beepdown
	4./* color */
		color
		color2

					a.
					b.
					c.
					d.
					e.
					f.
					g.