û�ж��ģ�
	1.GDT��ȫ���ڴ��--> 
		a.����������Ψһ��Ӧһ����ѡ���ӣ���������ͬ�ĶΣ������Ρ����ݶΡ���ջ�εȣ�
		b.��ѡ���ӣ�����������������G/L���е�λ�ã�+ ��־λ����GDT����LDT��
		c.��ʹ��ѡ����ǰ������GDTR/LDTR���Ĵ��������ҵ�������������εõ���ѡ���ӣ����ص���Ӧ�μĴ�����CS��DS��SS�ȣ�
		d.��ѡ���ӺͶμĴ��� ���-->cpuȷ���ڴ����λ�á���������-->����Ȩ�޺ͱ������
	(ok)2.p467 Ϊʲô������Ӧ���� 32MB
	3.p547 ˢ�¸���
	4.p600 map
	(ok)5.p649 time�Ż�
	6.int20 �Ķ�
	7.p925 ������led����
	8.p1160 cpu�Ż�ִ���쳣int
	9.p1263 linewin()

CALL �� JMP : call�����أ�jmp����ִ����ȥ

һ����������API�Ĺ���: (��ϣ�������һ���ַ�)
	ǰ��: ����һ��C���Ժ���void cons_putchar(struct CONSOLE *cons, int chr, char move)�������ҽ�hello.nasע�ᵽGDT��1003��
	1.������ʾ���ַ� 'A'->AL�Ĵ��� ����cons_putchar(x,x,x)��ʾ����
		need.ִ��CALL: ��Ҫ֪��cons_putchar(x,x,x)�����ĵ�ַ
		question.����cons_putchar(x,x,x)��C���Ա�д�����ն�ջ��������ݣ����ղ���AL�Ĵ���
	2.���Ǳ���Ҫһ��(����Ӧ�ó��򲿷֣������ڲ���ϵͳnaskfunc.nas���ֵ�)_asm_cons_putchar: PUSH EAX - PUSH (cons�ĵ�ַ)����hello.nas����CALL���_asmxx����
		need.�ҵ�cons�ĵ�ַ: ��δ���壬��������0x0fec(��ӵ�console_task��: ����*((int *) 0x0fec) = (int) &cons;)
		question.Ŀǰ��֪��_asm�����ĵ�ַ
	3.make�ҵ�haribote.map�������ҵ�����:"0x00000BE3 : _asm_cons_putchar"��API������Ҫʹ��Far-CALL����������Ϊ: CALL 2*8:0xbe3�����ڱ�F-Cָ������'RETF'
	4.����Ҫ��C��������д��������hello.nas�ĺ���
		need.C�����������farcall()
	5.��naskfunc.nas�������_far_call��ʹ�ã�C����: farcall(0, 1003*8)�����������޸��˲���ϵͳ�����룬_asm_cons_putchar��map���λ�ûᷢ���ı䣬Ҫ��hello.nas�����޸�Far-CALL��ַ
		need.��̬��_asmxx��ַ
	6.��IDT����ע�ắ����������Ϊ0x40�ţ�������init_gdtidt()���棬��Ҫset_gatedesc(idt + 0x40, (int)������:_asmxx, 2 * 8, AR_INTGATE32);��ʱ��hello.nasͨ��F-C����_asmxx��INT��ʽ��_asmxx��ҪIRETD���ء�
	*�ܽ�: 
	C����(�����ڻ��ʵ�ֵ�far_call()) 
	---> ͨ��GDT����hello.nas 
	---> ͨ��INT/IDT����nask.nas�����_asmxx���� 
	---> ͨ��PUSH 1/ PUSH EAX/ PUSH DWORD [0x0fec]/ ADD ESP,12/ ����������C���Ժ���

APP(�ļ���ʽ�洢����)ʵ�֣�
	��main() 
	--(����console.c)-> ��console 
	--(��������)-> ʶ��APP 
	--(��Ѱapp.c�ļ�)-> �ļ�main�������ù���|����:api_point() 
	--(��ջ��ֵ)--> ��a_nask.nas�ҵ���ຯ��_api_point() 
	--(����Ĵ�������ջȡֵ��INT40)-> �ҵ�naskfunc.nas��_asm_hrb_api() 
	--(CALL(console.c����ĺ���)hrb_api������Ĵ�������ջ��ȫ�Ĵ���)-> hrb_api()���ռĴ�����������Ӧʵ��xx���ܵĺ���|��:����һ�����ص�

�ڴ�ֲ���
	0x00000000 - 0x000fffff : ��Ȼ�������л���ʹ�ã���֮��ͱ�ա���1MB��
	0x00100000 - 0x00267fff : ���ڱ������̵����ݡ���1440KB��
	0x00268000 - 0x0026f7ff : �գ�30KB��
	0x0026f800 - 0x0026ffff : IDT ��2KB��
	0x00270000 - 0x0027ffff : GDT ��64KB��
	0x00280000 - 0x002fffff : bootpack.hrb��512KB��
	0x00300000 - 0x003fffff : ջ��������1MB��
	0x00400000 - laaaaaaast : ��

�Ĵ����ֲ���
	backlink: ָ��ǰһ��TSS��ָ�룬����ʵ�������л�ʱ������ṹ��
	esp0: �ں�ջ��ջ��ָ�룬���������л�ʱ���û�̬�л����ں�̬��---->�����ڴ�ռ䣬*((int *) (task_cons->tss.esp + 4)) = (int) sht_cons; //��CMDͼ���ַ�ŵ�CMD������ײ�
	ss0: �ں����ݶ�ѡ���ӣ����������л�ʱ���û�̬�л����ں�̬��
	esp1: ��ʱδʹ�á�
	ss1: ��ʱδʹ�á�
	esp2: ��ʱδʹ�á�
	ss2: ��ʱδʹ�á�
	cr3: ҳĿ¼��ַ�Ĵ�����Page Directory Base Register��PDBR�����洢�������ҳ����Ϣ��
	eip: �����ָ��ָ�룬��¼��������һ��Ҫִ�е�ָ���ַ�� ---->ѡ��Ҫִ�еĳ���Σ�task->tss.eipָ���������Ҫ���еĲ���
	eflags: ����ı�־�Ĵ������洢�˸��ֱ�־λ�����λ��־�����־�ȡ�
	eax, ecx, edx, ebx, esp, ebp, esi, edi: ͨ�üĴ��������ڴ洢�����ͨ�����ݡ�
	es, cs, ss, ds, fs, gs: ����Ķ�ѡ���ӣ�����ָ�������ڲ�ͬ�μĴ����еĶ���������
	ldtr: �ֲ���������Ĵ�����Local Descriptor Table Register�����洢��LDT�Ķ�ѡ���ӡ�
	iomap: I/Oλͼ��ַ���������������I/O�˿ڵķ���Ȩ�ޡ�

�ļ���¼��
	struct FILEINFO {
		unsigned char name[8]; 
			1.ǰ8���ֽ����ļ���������8���ֽ�ʱ���ÿո��㡣 
			2.���е��ļ������Ǵ�д�� 
			3.��һ���ֽ�0xe5���ļ��ѱ�ɾ����0x00���������κ��ļ�����Ϣ��
		unsigned char ext[3]; 
			1.����ļ�����3���ֽڣ���չ��
		unsigned char type; 
			1.һ���ֽڵ�������Ϣ
			2.ÿһλ�����⺬��
				0x01��ֻ���ļ�������д�룩
				0x02�������ļ�
				0x04��ϵͳ�ļ�
				0x08�����ļ���Ϣ������������Ƶȣ�
				0x10��Ŀ¼
		char reserve[10];
			1.10�ֽڱ���
		unsigned short time, date;
			1.2�ֽ�time �� 2�ֽ�date
		unsigned short clustno;
			1.2�ֽڣ��ļ����ݴӴ����ϵ��ĸ�������ʼ���
			2.clustno�����غš���cluster number��
			3.����ӳ���еĵ�ַ = clustno * 512 + 0x003e00
		unsigned int size;
			1.4�ֽڴ���ļ���С
	};


ipl10.nas �� ����boot����ļ����Դ��̼��ػ������� ��ʼ��0x7c00��


asmhead.nas �� �����ˣ�
			1.ͼ�ηֱ������� ���Ƿ�VBE��
			2.��ȡ����led״̬��INT16�� 
			3.��ֹ�����жϣ�ͨ��CLI����pic��
			4.����A20��ַ�ߣ�Ϊ��cpu����1mb���ϵ��ڴ�ռ䣩
			**5.����GDT������GDTR��cr0-->ϵͳģʽ��
			*6.���������ݺ�bootpack��.hrb�����Ƶ��ڴ棨memcoy([esi]Դ,[edi],[ecx])��
			7.waitkbdout
			8.����bootpack
            �Ŀ���������������ʼ��0xc200
               �ص��ڣ�


naskfunc.nas �� ���ڽ��C�����޷��������ϵӲ�����⣬���岿��C��������Ҫ�ĺ����������ˣ�
	_io_hlt: ִ�� HLT ָ�ʹ CPU ����ͣ��״̬��
	_io_cli: ִ�� CLI ָ���ֹ�ⲿ�жϡ�
	_io_sti: ִ�� STI ָ������ⲿ�жϡ�
	_io_stihlt: ִ�� STI ָ�Ȼ��ִ�� HLT ָ������ⲿ�жϲ�ʹ CPU ����ͣ��״̬��
	_io_in8: ��ָ���� I/O �˿ڶ�ȡһ���ֽڵ����ݡ�
	_io_in16: ��ָ���� I/O �˿ڶ�ȡһ���֣�16 λ�������ݡ�
	_io_in32: ��ָ���� I/O �˿ڶ�ȡһ��˫�֣�32 λ�������ݡ�
	_io_out8: ��һ���ֽڵ�����д��ָ���� I/O �˿ڡ�
	_io_out16: ��һ���֣�16 λ��������д��ָ���� I/O �˿ڡ�
	_io_out32: ��һ��˫�֣�32 λ��������д��ָ���� I/O �˿ڡ�
	_io_load_eflags: ��ȡ EFLAGS �Ĵ�����ֵ��
	_io_store_eflags: ��ָ����ֵ�洢�� EFLAGS �Ĵ�����
	_load_gdtr: ����ȫ����������GDT�������ƺͻ���ַ��
	_load_idtr: �����ж���������IDT�������ƺͻ���ַ��
	_load_tr: 
	_asm_inthandler21: �жϴ�����򣬴��� IRQ 0x21 �жϡ�
	_asm_inthandler27: �жϴ�����򣬴��� IRQ 0x27 �жϡ�
	_asm_inthandler2c: �жϴ�����򣬴��� IRQ 0x2C �жϡ�
	_farjmp: ��������ת����ִ��JMP FAR��


C�����ļ��У�
	1.bootpack.h ͷ�ļ�
		1./* asmhead.nas */
			struct BOOTINFO(��Ļ��ز���):
					a.cyls-���棨��ͷ����Ŀ
					b.leds-����led
						{
							MOV BX,0x4101 ;VBE��640x480x8bi��ɫ:103,105,107�ֱ��������ķֱ���
							MOV AX,0x4f02 ;1.����VBEģʽ��AX = 0x4f02��BX = ����ģʽ����
										   2.������VBEģʽ��AH =0��AL=����ģʽ����
							INT 0x10
						}
					c.vmode-��ʾģʽ���ı���ͼ�Σ�
					d.reserve-����
					e.scrnx scrny-��Ļ�ķֱ��ʣ���Ⱥ͸߶ȣ���������
					f.*vram-��Ļ��ʾ�ڴ����ʼ��ַfirst

		2./* naskfunc.nas */ �����溯����
			void io_hlt(void); ͣ��
			void io_cli(void); ��ֹ�ж�
			void io_sti(void); �����ж�
			void io_stihlt(void); STI && HLT
			int io_in8(int port); ��port���˿ڵ�ַ����ȡһ��8λ����
			void io_out8(int port, int data); ��dataд��port
			int io_load_eflags(void); ��ȡeflags��32λ��CF��λ��PF��ż��AF������λ��ZF�㡢SF���š�OF�����
			void io_store_eflags(int eflags); �洢һ��ֵ��eflags
			void load_gdtr(int limit, int addr); addrΪ��ַlimitΪ���ޣ�����GDT�Ĵ�����GDTR��
			void load_idtr(int limit, int addr); ͬ������IDT�Ĵ���
			int load_cr0(void); ��ȡCR0��ֵ
			void store_cr0(int cr0); ��cr0��ֵ����CR0�Ĵ���
			void load_tr(int tr); cpu��ס��ǰ���е����񣬡�taskregister��������Ĵ��������������(gdt+n) -> tr = n * 8;
			void asm_inthandler20(void); ��ʱ���ж�
			void asm_inthandler21(void); �����ж�
			void asm_inthandler27(void); �����ж�
			void asm_inthandler2c(void); ����ж�
			unsigned int memtest_sub(unsigned int start, unsigned int end); �����ڴ棬��start��end
			void farjmp(int eip, int cs); [ESP+4]���λ�þʹ����eip��ֵ��[ESP+8]�����cs��ֵ
			void farcall(int eip, int cs); ��farjmp����
			void asm_hrb_api(void); 
			void start_app(int eip, int cs, int esp, int ds, int *tss_esp0);
			void asm_end_app(void); ����app�ص��ʼ��ȡapp����ʱ

		3./* fifo.c */: first:mousefifo, keyfifo, timerfifo / second: fifo
			struct FIFO8(�ֽ��ͻ�����): #����һ��FIFO32�����Ƶģ�ֻ��char���ͺ�int���͵�����
					a.*buf-���仺��������ʼ��ַ
					b.p-��һ����д�����ݵ�����
					c.q-��һ���ɶ�ȡ���ݵ�����
					d.size-�������ܴ�С
					e.free-������ʣ��ռ��С
					f.flags-��־λ��״̬��Ϣ

			void fifo32_init(struct FIFO8 *fifo, int size, unsigned char *buf); ��ʼ��ѭ����������������Ҫ��size��*buf������һ��*fifo
			int fifo32_put(struct FIFO8 *fifo, unsigned char data); д��һ��data���ڣ�*buf+p�������ж�full�������������ʱ���ߣ���������
			int fifo8_get(struct FIFO8 *fifo); ��ȡ���� data or -1
			int fifo8_status(struct FIFO8 *fifo); ����ȡfree

		4./* graphic.c */ ����Ļ���ơ�
			#define����ĸ�����ɫ���Ӧֵ
			void init_palette(void); init��ɫ�壬��ʼ��һ����ɫ
			void set_palette(int start, int end, unsigned char *rgb); set��ɫ�壬���У�0x03c8�ǵ�ɫ��������Ĵ�����0x03c9�ǵ�ɫ������ݼĴ���������������char����һ����ɫ
			void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1); �ھ��Σ�x0,x1,y1,y0�����������ɫc��vram-������صĻ���������Ļ�׵�ַ����xsize-�п��
			void init_screen8(char *vram, int x, int y); init��Ļ����Ϊ������ɫ
			void putfont8(char *vram, int xsize, int x, int y, char c, char *font); �ڣ�x��y���������ַ�*font������ɫc
			void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s); ����putfont8����ʵ�ֻ����ַ���
			void init_mouse_cursor8(char *mouse, char bc); *mouse�������ɫ�飬������ɫbc���������
			void putblock8_8(char *vram, int vxsize, int pxsize,int pysize, int px0, int py0, char *buf, int bxsize); ����һ����pxsize��pysize��ͼ��飻��*buf������ݣ�����Ϊbxsize

		5./* dsctbl.c */ ���жϱ���ļ������
			struct SEGMENT_DESCRIPTOR(��������):
					a.short limit_low, base_low;
					b.char base_mid, access_right;
					c.char limit_high, base_high;
			struct GATE_DESCRIPTOR(��������):
					a.short offset_low, selector;
					b.char dw_count, access_right;
					c.short offset_high;

			void init_gdtidt(void); init gdt��idt | ����(���INT)����
			void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar);
			void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar);

		6./* int.c */ ���жϡ�
			#define������PIC_xx���ֲ�����
			void init_pic(void); init-pic����ICW1-4�� pic0��IRQ0--7��INT20--27����pic1��������pic0��IRQ2�ϣ�IRQ8--15��INT28--2f��
			void inthandler27(int *esp); INT27�ɵ����������𣬲���Ҫ������ֻ��֪ͨ

		7./* keyboard.c */ �����̡�
		#define������KEY��DTA|CMD��
			void inthandler21(int *esp); INT21
			void wait_KBC_sendready(void);for(;;)����ѭ�� ����Ƿ�׼���ý������ݣ�KEYSTA_SEND_NOTREADY == 0 ��ʾ����
			void init_keyboard(void); init����
			extern strtuct FIFO8 keyfifo; ����һ��ȫ��keyfifo������

		8./* mouse.c */ ����꡷
			struct MOUSE_DEC(���״̬��Ϣ):
					a.buf[3]-char������������ֽ�
					b.phase-����״̬
					c.x y-ˮƽ����ֱ״̬
					d.btn-��� ��ť״̬

			void inthandler2c(int *esp); INT2c
			void enable_mouse(struct MOUSE_DEC *mdec); ������꣬����*mdec�����������Ϣ��phase�����洢��*mdec�з���
			int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat); ����*mdec�е����ݣ�������*mdec��dat����Դ��mousefifo�����������ֵ������& |����Ϻ����*buf
			extern struct FIFO8 mousefifo; ����ȫ��mousefifo������

		9./* memory.c */ ���ڴ桷
			struct FREEINFO(��¼�ڴ�):
					a.addr-��ʼ��ַ
					b.size-�ڴ���С
			struct MEMMAN(�ڴ������):
					a.free-��ǰ�����ڴ������
					b.maxfrees-��������ڴ������
					c.lostsize-�ڴ治�㶪�����ڴ���ܴ�С
					d.losts-�������ڴ������
					e.free[]-freeinfo�ṹ������

			unsigned int memtest(unsigned int start, unsigned int end); test�����ڴ��ٷ�Χ��start��end�������汾386or486���޸�cr0
			void memman_init(struct MEMMAN *man); init-mamman
			unsigned int memman_total(struct MEMMAN *man); memmanָ��ǰ�������ڴ�
			unsigned int memman_alloc(struct MEMMAN *man, unsigned int size); memman�����һ��size��С���ڴ�
			int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size); ��memman�ͷ�addr����Сsize���ڴ棨�����ڴ�ϲ���
			unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size); memman_alloc x 4k :��size = (size + 0xfff) & 0xfffff000��
			int memman_free_4k(struct MEMMAN *man, unsigned int addr, unsigned int size); memman_free x 4k

		10./* sheet.c */ ��ͼ����ơ�
			struct SHEET(һ��ͼ��/����):
					a.*buf-ͼ�����ػ�����
					b.bxsize-���
					c.bysize-�߶�
					d.vx0 vy0-(vx0, vy0)��ͼ������Ļ�ϵ�����
					e.col_inv-͸��ɫ
					f.height-ͼ��ĸ߶ȣ����ӣ�
					g.flags-ͼ���־λ
					h.*ctl-ָ��struct SHTCTLͼ�����λ��������������������������Ҫ����SHTCTL��
					i.*task-(struct TASK)���������ǲ��ǽ�Ҫ������app����: app����taskʱҪ�ر�sheetͼ�㡣sht->task = 0;���Զ��ر�
			struct SHTCTL(ͼ�������):
					a.*vram-�����ڴ棨��Ļ�׵�ַ��
					b.*map-ͼ�������ص���״̬
					c.xsize-���
					d.ysize-�߶�
					e.top-��ǰͼ����߸߶�
					f.*sheets[]-ÿ��Ԫ�ض���ָ�룬ָ��һ��ͼ�㣨sheets0�����һ�������˳������
					g.sheets0[]-���ڴ���sheetͼ�㣬�����������

			struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize); init-shtctl
			struct SHEET *sheet_alloc(struct SHTCTL *ctl); ����һ��ͼ��sheet
			void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv); ����ͼ��sht������buf����xsize����ysize��͸��ɫcolinv
			void sheet_updown(struct SHEET *sht, int height); ����sht->ctl->sheet[]->height = height
			void sheet_refresh(struct SHTCTL *ctl); ˢ��ҳ�棨��仯�޸��Ż���
			    ***a.sheet_refreshmap��ˢ��map��
				b.sheet_refreshsub��ˢ��sub��
			void sheet_slide(struct SHEET *sht, int vx0, int vy0); ��������sht����vx0��vy0��
			void sheet_free(struct SHEET *sht); �ͷ�sht����ʹ��sht

		11./* timer.c */ ����ʱ����
			struct TIMER
					e.*next-next_timerָ�룬ָ����һ��TIMER����һ������
					a.timeout-��ʱʱ�䣨��timerctl.count�Ƚϳ�ʱ��
					b.flags-��ʱ���Ƿ�ʹ��
					f.flags2-�Ƿ���Ӧ�ó����ʱ�����Ƿ���Ҫȡ��
					c.fifo-��ʱʱ��data���fifo
					d.data-��¼��ʱ
			struct TIMERCTL
					a.count-����ÿ��100�Σ��жϷ���������ϵͳ����ʱ�䣩
					b.next-next_timeout��һ����timeout��TIMER��ʱ��
					c.using-����ʹ�õļ�ʱ������
					(delete)*d.*timers[]-�洢���еĶ�ʱ����STRUCT TIMER����ÿ����ָ��һ����ʱ������˳������(Timer�������*next������Ҫ*timers[]);ȡ����֮�� *t0 --> *head
					(new)*d.*t0-*head
					e.timers0[]-�洢ʵ�ʶ�ʱ�����ݣ�STRUCT TIMER��

			void init_pit(void); ��ʼ��pit��������һ���ڱ�����󣬱�����ȫ��ʱ��
			struct TIMER *timer_alloc(void); ����һ��timer����עΪ��ʹ��
			void timer_free(struct TIMER *timer); flag = 0���ͷ�
			void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data); ��ʼ��timer���йص�*fifo��data
			void timer_settime(struct TIMER *timer, unsigned int timeout); ���ó�ʱʱ�䣬���������򣨾��Գ�ʱʱ���С�������뵽������, timeout-->0.01s��λ
			void inthandler20(int *esp); int20��IRQ0
			int timer_cancel(struct TIMER *timer); ȡ��ָ����ʱ��
			void timer_cancelall(struct FIFO32 *fifo); ���� ȡ��/�ͷ� ��ʱ��
			
			extern struct TIMERCTL timerctl;

		12./* mtask.c */ ��������
			struct TSS32
					���ϡ��Ĵ�����
			struct TASK ����tcb
					a.sel-���GDT��ţ�ѡ���ӣ�
					b.flags-����������û������
					c.level-����ȼ�
					d.priority-�������ȼ�
					e.fifo-������
					f.tss-(struct TSS32)
					g.*cons-(struct CONSOLE)ָ������cons[x]��һ���ն�
					h.ds_base-Ӧ�����ݶε�ַ
			struct TASKLEVEL
					a.running-��Levelӵ�е�Task����
					b.now-�������е�Task
					c.*tasks-(struct TASK)��Level����������
			struct TASKCTL
					a.now_lv-���ڵ�Level
					b.lv_change-��һ���л�����ʱ�費��Ҫ�л�Level
					c.level[]-(struck TASKLEVEL)���ڵ�level���б�
					d.tasks0-(struck TASK)���ڵ�Task���б�

			//�ڲ�
			void task_add(struct TASK *task); ����*task��Level
			void task_remove(struct TASK *task); �Ƴ�*task���ж�λ��+����taskΪ˯��
			void task_switchsub(void); taskctl->now_lv�ҵ����ȼ���ߵ�level(0--10)��Ȼ���л��������
			void task_idle(void); �ڱ�Task
			//����ӿ�
			struct TASK *task_now(void); ȡ��Level->tasks[Level->now]
			struct TASK *task_init(struct MEMMAN *memman); init����Task��������һ�����Level���ڱ�
			struct TASK *task_alloc(void); ����һ��Task����ʼ��TSS
			void task_run(struct TASK *task, int level, int priority); ָ����һ�����е�*task��������װ��level���ָ��priority
			void task_switch(void); �л�����
			void task_sleep(struct TASK *task); ˯�ߣ�����˯�ߣ��л����񣩡�����˯�ߣ�
			extern struct TIMER *mt_timer;

		13./* window.c */ �����ڡ�
			void make_window8(unsigned char *buf, int xsize, int ysize, char *title, char act); ����һ��������ͼ�����ݴ洢����bufָ����ڴ��У����xsize���߶�ysize������title�����ڵĸ���Ԫ�ص����
			void putfonts8_asc_sht(struct SHEET *sht, int x, int y, int c, int b, char *s, int l); ���� boxfill8����������ɫ����putfonts8_asc�������ַ�������sheet_refresh��ˢ��ͼ��ҳ�棩 ��������
			void make_textbox8(struct SHEET *sht, int x0, int y0, int sx, int sy, int c); ����һ���ı�����*shtͼ�����棬����ڣ�x0��y0�����Կ�sx����sy��������ɫc����
			void make_wtitle8(unsigned char *buf, int xsize, char *title, char act); ���ƴ��ڵı��������ݣ�������ı��ͻ���ڱ�־��act�����Ƿ񲻱�ѡ�б��
			
		14./* console.c */ ���նˡ�
			void console_task(struct SHEET *sheet, unsigned int memtotal); �������ն�-�����񣬰������������mem��clr��dir��type����sheet���ƴ��ڣ�memtotal���ڴ�
			int cons_newline(int cursor_y, struct SHEET *sheet); ˢ���µ�һ�У����滻�к͹�������������cursor_y�������µ�����cursor_y
			void cons_putchar(struct CONSOLE *cons, int chr, char move); 1.���ж��Ƿ�Ϊ���Ʊ�����س��������з��� 2.��ͨ���ַ�������putfonts8_asc_sht�����ڹ��λ��������ַ���move ��= 0 �ƶ����8�����ء� 
			void cons_putstr0(struct CONSOLE *cons, char *s); ���յ�0�������ַ������뷨
			void cons_putstr1(struct CONSOLE *cons, char *s, int l); ���߳���Ϊl���ַ������뷨
			void cons_runcmd(char *cmdline, struct CONSOLE *cons, int *fat, unsigned int memtotal); ������men��dir��cls��type����ϵͳcmd����ĵ���
			void cmd_mem(struct CONSOLE *cons, unsigned int memtotal); ��ֳ�����mem����
			void cmd_cls(struct CONSOLE *cons); cls����
			void cmd_dir(struct CONSOLE *cons); dir����
			void cmd_type(struct CONSOLE *cons, int *fat, char *cmdline); type����
			int cmd_app(struct CONSOLE *cons, int *fat, char *cmdline); ��ȥϵͳcmd���������APP/API�����fat����Ѱ�������ļ���
				(�������Ρ����ݶε�ַ)
			void hrb_api(int edi, int esi, int ebp, int esp, int ebx, int edx, int ecx, int eax); ִ��api��ֵ������Ĵ�����ֵ���ж�EDX��ֵ�ж�����
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
				19.)��һ����������
			int *inthandler0d(int *esp); 
			int *inthandler0c(int *esp);
			void hrb_api_linewin(struct SHEET *sht, int x0, int y0, int x1, int y1, int col);

		15./* file.c */ ���ļ�������ļ�������FAT�Ĺ�ϵ��
			struct FILEINFO
					�����ļ���¼
			void file_readfat(int *fat, unsigned char *img); ��ѹ���ļ�������FAT���洢������fat��
			void file_loadfile(int clustno, int size, char *buf, int *fat, char *img); ���ļ�ϵͳ�м����ļ����� ���������С��ҵ�clustno�������ļ�size��С��buf��ָ��洢�ļ����ݵĻ�������ָ�롣fat���ļ������File Allocation Table����ָ�롣img����ʾ�����ļ�ϵͳӳ����ַ������ָ�롣
			struct FILEINFO *file_search(char *name, struct FILEINFO *finfo, int max);

	2.dsctbl.c gdt���ֶΣ���idt���жϣ�
	3.graphic.c ��Ļ��ʾ�����桢���ı�־
	4.int.c �жϣ�!21��27��!2c��pic0��pic1
	5.keyboard.c ���̣�21int��
	6.mouse.c ��꣨2cint��
	7.memory.c �ڴ棨menman�ڴ����   
	8.sheet.c ͼ�����
	9.timer.c ��ʱ��
	10.mtask.c ������

	last_but_important.bootpack.c �����ܽ���
		
		�޸ĳ�cons[0] cons[1]
		HariMain()-->fifo_buf
			1.256~511ȷ���Ǽ�������
				1.256~256+0x80�Ǽ��̵�ɨ���루���õİ���������������Ϊ0
				2.A~Z��a~z��ͨ�� key_leds �� key_shift �任
				3.���� key_to(�ж����ĸ�����) ��ȡs[0]
				4.256 + 0x0e �˸�
				5.256 + 0x1c �س�
					�����Ƿ���������ִ������
				6.256 + 0x0f tab-�л�����
				7.256 + 0x2a ��shift���£�key_shift��һλ��Ϊ1---256 + 0xaa ��shift�ͷţ�key_shift��һλΪ��Ϊ0
				8.256 + 0x36 ��shift���£�key_shift�ڶ�λ��Ϊ1---256 + 0xb6 ��shift�ͷţ�key_shift�ڶ�λΪ��Ϊ0
				9.256 + 0x3a CapsLock
				10.256 + 0x45 NumLock
				11.256 + 0x46 ScrollLock
				12.256 + 0xfa ����ȷ�Ͻ������ݣ�key_wait��Ϊ-1
				13.256 + 0xfe ����ʧ�ܣ����·�������
				14.
		1.�ڴ��鴦����ת���last��
		2.����
		3.for����жϣ����-���ƶ����϶�win���ڣ�
					������-����ʾa�루ת���ɴ�С��ĸ/����/���֣�����ȡkeytable[]-��ʾABC��shift��Lock&Led���س���
					����ʱ��-����ʱ����꣨��cursor_c=-1��ʾ����𣩣�
					���������л�-��TSS��д��TCB�������񴥷����̣�
					�����ڹ���������ÿ�����ؿ��������һ�е����ؿ飩
					������console_task���������棩mem��dir��clsָ�

		void keywin_off(struct SHEET *key_win); �ر�ָ�����ڵı��⣬�����ݴ��ڵ�flags���Ծ����Ƿ�����ô��ڹ����������FIFO�����з���ֵ3���Կ��ƿ���̨�Ĺ��ر�
		void keywin_on(struct SHEET *key_win); ��ָ������


app�ļ�: 
	1./* a.nas */ ��Ϊhrb_api()׼����api�����ӿڡ�7
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