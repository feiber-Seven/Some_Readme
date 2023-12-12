1. hello驱动: 
    hello_drv.c--main: 

        int major;


        struct file_operations hello_drv = {
            .owner	 = THIS_MODULE,
            .open    = hello_drv_open,
            .read    = hello_drv_read,
            .write   = hello_drv_write,
            .release = hello_drv_close,
        };


        char kernel_buf[1024];
        ssize_t hello_drv_read (struct file *file, char __user *buf, size_t size, loff_t *offset){
            copy_to_user(buf, kernel_buf, MIN(1024, size)); /* buf[]是user的、本文件定义kernel_buf[1024] */
        }
        ssize_t hello_drv_write (struct file *file, const char __user *buf, size_t size, loff_t *offset){
            copy_from_user(kernel_buf, buf, MIN(1024, size)); /* buf[]是user的、本文件定义kernel_buf[1024] */
        }
        int hello_drv_open (struct inode *node, struct file *file)
        int hello_drv_close (struct inode *node, struct file *file)


        struct class *hello_class;
        int __init hello_init(void){
            register_chrdev(0, "hello", &hello_drv);  /* 注册hello_drv结构体为/dev/hello */
            class_create(THIS_MODULE, "hello_class"); /* 创建class结构体 */ ？干嘛用的呢
            device_create(hello_class, NULL, MKDEV(major, 0), NULL, "hello"); /* 创建设备节点MKDEV(主设备号, 次设备号)名字“hello” /dev/hello */
        }
        void __exit hello_exit(void){
            device_destroy(hello_class, MKDEV(major, 0));
            class_destroy(hello_class);
            unregister_chrdev(major, "hello");
        }


        module_init(hello_init); /* 创建告诉内核入口 */
        module_exit(hello_exit);

    补充:
        1.kconfig: 提供界面menuconfig内容
        2..config: 保存配置值
        3.Makefile: 使用配置值

2. 简单的led控制:
    led_drv.c--main:


3. 分离出简单的board文件:
    led_drv.c--main:

        int major;
        struct led_operations *p_led_opr;

        struct file_operations led_drv = {
            .owner	 = THIS_MODULE,
            .open    = led_drv_open,
            .read    = led_drv_read,
            .write   = led_drv_write,
            .release = led_drv_close,
        };

        ssize_t led_drv_read (struct file *file, char __user *buf, size_t size, loff_t *offset)
        ssize_t led_drv_write (struct file *file, const char __user *buf, size_t size, loff_t *offset){
                struct inode *inode = file_inode(file); /* 从文件中获取inode节点 */
	            int minor = iminor(inode); /* 从结构体中获取次设备号 */

                copy_from_user(&status, buf, 1); /* 只需要0/1 */
                p_led_opr->ctl(minor, status); /* 调用control函数(次设备号，状态) */
        }
        int led_drv_open (struct inode *node, struct file *file){
            int minor = iminor(node);
            p_led_opr->init(minor); /* 打开时初始化设备 */
        }
        int led_drv_close (struct inode *node, struct file *file)

        struct class *led_class;
        int __init led_init(void){
            register_chrdev(0, "100ask_led", &led_drv);  /* /dev/led */
            led_class = class_create(THIS_MODULE, "100ask_led_class");

            /* 得到结点并创建节点、创建多个LED */
            p_led_opr = get_board_led_opr();
            for (i = 0; i < p_led_opr->num; i++)
		        device_create(led_class, NULL, MKDEV(major, i), NULL, "100ask_led%d", i); /* /dev/100ask_led0,1,... */
        }
        void __exit led_exit(void){
            /* 每一个创建的LED都销毁掉 */
            for (i = 0; i < p_led_opr->num; i++)
                device_destroy(led_class, MKDEV(major, i)); /* /dev/100ask_led0,1,... */

            device_destroy(led_class, MKDEV(major, 0));
            class_destroy(led_class);
            unregister_chrdev(major, "100ask_led");
        }

        module_init(led_init);
        module_exit(led_exit);
        MODULE_LICENSE("GPL");


    board_led.c--main:

        struct led_operations board_demo_led_opr = {
            .num  = 1,
            .init = board_demo_led_init,
            .ctl  = board_demo_led_ctl,
        };

        struct led_operations *get_board_led_opr(void)


    led_opr.h--main:
        struct led_operations {
            int num; /* how much */
            int (*init) (int which); /* which_LED */
            int (*ctl) (int which, char status); /* which_LED what_status_0/1 */
        };


4. 分离驱动、芯片、硬件设备:{
        board_A_led.c /* 独立设备: 使用led_resource表明GPIOx_y */
        chip_demo_gpio.c /* 芯片管理: 实现led_operations并关联led_resource */
        led_drv.c /* 上层驱动开发: 调用led_operations将用户open、write联系init、crl */
        led_resource.h /* 定义'struct led_resource'决定设备编号-x组y号引脚 */
        led_opr.h /* 定义'struct led_operations'对设备init、control */
}

    board_A_led.c--mian:

        struct led_resource board_A_led /* 装填结构体 */

        led_resource *get_led_resouce(void) /* 实现函数 */


    chip_demo_gpio.c--main:

        struct led_resource *led_rsc

        struct led_operations board_demo_led_opr = {
        } /* 定义led_operations结构体 */

        int board_demo_led_init (int which){
            led_rsc = get_led_resouce(); /* 从board_A_led.c中取出resource */
        }
        int board_demo_led_ctl (int which, char status)

        struct led_operations *get_board_led_opr(void) /* 实现get_board_led_opr */


    now.led_drv.c 和 3.led_drv.c 一模一样


5. bus总线版: {
    board_A_led.c --> platform_device
    chip_demo_gpio.c --> platform_driver
    led_drv.c
}分别需要编译出.ko文件，都有init函数和exit函数

    led_opr.h 4.分离 时一致
        struct led_operations {
            int (*init) (int which);
            int (*ctl) (int which, char status);
        };

    
    led_resource.h:
        #define GROUP(x) (x>>16)
        #define PIN(x)   (x&0xFFFF)
        #define GROUP_PIN(g,p) ((g<<16) | (p))


    led_drv.h: /* 声明led_drv.c里的函数 */
        void led_class_create_device(int minor);
        void led_class_destroy_device(int minor);
        void register_led_operations(struct led_operations *opr);


    board_A_led.c--main:

        /* platform_device结构体实现 */
        struct platform_device board_A_led_dev = {
            .name = "100ask_led",
            .num_resources = ARRAY_SIZE(resources),
            .resource = resources,
            .dev = {
                    .release = led_dev_release,
            },
        };
        其中的resource:
            struct resource resources[] = {
                {
                        .start = GROUP_PIN(3,1),
                        .flags = IORESOURCE_IRQ,
                        .name = "100ask_led_pin",
                },
                {
                        .start = GROUP_PIN(5,8),
                        .flags = IORESOURCE_IRQ,
                        .name = "100ask_led_pin",
                },
            };

        int __init led_dev_init(void){
            platform_device_register(&board_A_led_dev);
        }
        int __exit led_dev_exit(void){
            platform_device_unregister(&board_A_led_dev);
        }

        module_init(led_dev_init);
        module_exit(led_dev_exit);


    chip_demo_gpio.c--main:

        /* led_operations结构体实现 */
        struct led_operations board_demo_led_opr = {
            .init = board_demo_led_init,
            .ctl  = board_demo_led_ctl,
        };

        int board_demo_led_init (int which)       
        int board_demo_led_ctl (int which, char status)

        struct led_operations *get_board_led_opr(void)

        /* platform_driver结构体的实现 */
        struct platform_driver chip_demo_gpio_driver = {
            .probe      = chip_demo_gpio_probe,
            .remove     = chip_demo_gpio_remove,
            .driver     = {
                .name   = "100ask_led", /* 匹配的name标志 */
            },
        };

        int g_ledpins[100]; 100个灯数组
        int g_ledcnt = 0; 统计有多少个灯
        int chip_demo_gpio_probe(struct platform_device *pdev){
            struct resource *res;
            int i = 0;
            while (1){
                res = platform_get_resource(pdev, IORESOURCE_IRQ, i++);
                if (!res)
                    break;
                g_ledpins[g_ledcnt] = res->start;
                led_class_create_device(g_ledcnt);
                g_ledcnt++;
            }
            return 0;
        }
        int chip_demo_gpio_remove(struct platform_device *pdev){
            struct resource *res;
            int i = 0;
            while (1)
            {
                res = platform_get_resource(pdev, IORESOURCE_IRQ, i);
                if (!res)
                    break;
                led_class_destroy_device(i);
                i++;
                g_ledcnt--;
            }
        }

        static int __init chip_demo_gpio_drv_init(void){
            platform_driver_register(&chip_demo_gpio_driver); 
            register_led_operations(&board_demo_led_opr);
        }
        static void __exit lchip_demo_gpio_drv_exit(void){
            platform_driver_unregister(&chip_demo_gpio_driver);
        }

        module_init(chip_demo_gpio_drv_init);
        module_exit(lchip_demo_gpio_drv_exit);


    led_drv.c--main:

        void led_class_create_device(int minor){
            /**  
             * 将对具体device设备操控封装起来，供下层使用
             * 因为led_class和major在本层定义，而具体设备由下层指出minor（led0/led1/...）
             */
            device_create(led_class, NULL, MKDEV(major, minor), NULL, "100ask_led%d", minor); /* /dev/100ask_led0,1,... */
        }
        void led_class_destroy_device(int minor){
            /* 封装 */
            device_destroy(led_class, MKDEV(major, minor));
        }
        void register_led_operations(struct led_operations *opr){
            /* 将下层的*opr传到本层led_drv.c里面调用 */
            p_led_opr = opr;
        }

        EXPORT_SYMBOL(led_class_create_device);
        EXPORT_SYMBOL(led_class_destroy_device);
        EXPORT_SYMBOL(register_led_operations);

        struct file_operations led_drv = { /* 供用户调用的file_operations接口和之前保持一致 */
            .owner	 = THIS_MODULE,
            .open    = led_drv_open,
            .read    = led_drv_read,
            .write   = led_drv_write,
            .release = led_drv_close,
        };

        ...

        int __init led_init(void){
            /* 省略了device_create()步骤，交给下层(芯片层chip)处理 */
        }
        void __exit led_exit(void){
            /* 省略device_destroy() */
        }

        module_init(led_init);
        module_exit(led_exit);


6. 设备树版: 
    设备树: 
        dts(设备树源文件(dtsi为设备树头文件)) --dtc--> dtb
        /des-v1/ -->版本
        [memory reservations]  -->格式为/memreserve/ <address> <length>
        /{
            [label:] node-name[@unit-address]{
                [properties definition]
                [child nodes]
            }

            cpu{
                name = val -->格式"string" <v32> <12>
                xxx
            };

            #address-cells = <1>; -->几个数据表示地址
            #size-cells =<1>; -->几个数据表示大小
            memory{
                reg = <0x80000000 0x20000000>
            }

            led{
                compatible = "A", "B", "C"; -->兼容
                model = "D"; -->兼容ABC但是我是D
                status = "ok" -->或者是"disable"
            }
        };

        在根节点外 使用/修改:
            &lebal{
                xxx
            }
            &{/node-name@unit-address}{
                xxx
            }


7. 按键控制: (不需要直接操作寄存器)
    pinctrl:
        1.在设备树里面定义一个UART节点(使用pinctrl，成为client_device)
            UART{
                /* state: xxx -- group: yyy1, yyy2 -- function: zzz */
                pinctrl-names = "defualt", "sleep";
                pinctrl-0 = <&xxx_0>;
                pinctrl-1 = <&xxx_1>; /* xxx-0 和 xxx_1 位于pinctrl里面 */
                status = "okay";
            }
        2.创建一个pincontroller节点:
            pincontroller{
                xxx_0{ /* 复用节点 */
                    function = "uart";
                    groups = "", "";
                } /* 默认状态时uart */

                xxx_1{ /* 配置节点 */
                    groups = "", "";
                    output-high；
                } /* sleep时配置为高电平 */
            }

    GPIO: (gpiod: new, gpio: old)
        1.在设备树里面创建device节点:
            device{
                led-gpio = <A, xx, flag>;
            }
        2.设备树里面另一个节点(含有属性gpio-controller):
            gpioA: gpio@xxx{
                gpio-controller;
                #gpio-cells = <2>;
            }

    INT:
        1.设备树创建需要用中断的设备:
            device{
                interrupt-parent = <&gpioA>;
                interrupt = <xx, type>; /* xx(hwirq)属于domain*/
            }
        2.使用irq_domain(hwirq->soft_irq)
            struct irq_domain{
                struct irq_domain_ops{
                        .map
                        .xlate -->解析得到irq(虚拟中断号)-irq_request
                        ......
                    }
                int linear_revmap[] -->(hwirq, irq)
                ......
            }
        3.设备树里面另一个节点(含有属性interrupt-controller)
            top_interrupt{
                compatible = "", "";
                interrupt-controller;
                #interrupt-cells = <2>;
            }
            next_interrupt{
                compatible = "", "";
                interrupt-controller;
                #interrupt-cells = <3>;

                interrupt-parent = <&top_interrupt>
                interrupts = <>, <>, <>; /* 使用parent里面一个或多个中断，描述每一个<>里的中断由parent->cells = <?>决定 */
            }

    中断GPIO:
        1.看原理图 --> 确定哪个GPIO作为按键中断
        2.在设备树添加节点 --> xxx_gpio_key{
                                    compatible = "xxx_gpio_key";
                                    gpios = <&gpioA x xxxx
                                                &gpioB y yyyy>;
                                    pinctrl-names = "default";
                                    pinctrl-0 = <&keyA_pinctrl
                                                    &keyB_pinctrl>;
                                }
                            /* 无需指定中断: gpiod_to_irq()转化成中断 */
        3.构造platform_driver, 在.pr obe里面获取gpio, 转化成irq
        4.request_irq

8. 按键设备gpio中断程序:
    1.定义platform_driver
        static const struct of_device_id ask100_keys[] = {
            { .compatible = "100ask,gpio_key" },
            { },
        };
        static struct platform_driver gpio_keys_driver = {
            .probe      = gpio_key_probe,
            .remove     = gpio_key_remove,
            .driver     = {
                .name   = "100ask_gpio_key",
                .of_match_table = ask100_keys, //设备节点的compatible
            },
        };
    
    2.出入口函数:
        static int __init gpio_key_init(void)
        static void __exit gpio_key_exit(void)

    3.实现gpio_key_probe、gpio_key_remove
        struct gpio_key{
            int gpio;
            struct gpio_desc *gpiod;
            int flag;
            int irq;
        };
        static struct gpio_key *gpio_keys_100ask;
        
        int gpio_key_probe(struct platform_device *pdev){
            count = of_gpio_count(node); //确定几个gpio
            gpio_keys_100ask = kzalloc(sizeof(struct gpio_key) * count, GFP_KERNEL);
            
            for (i = 0; i < count; i++){
                gpio_keys_100ask[i].gpio = of_get_gpio_flags(node, i, &flag); //获取第i个gpio并保存flag
                /* 处理保存引脚 */
                gpio_keys_100ask[i].gpiod = gpio_to_desc(gpio_keys_100ask[i].gpio);
                gpio_keys_100ask[i].flag = flag & OF_GPIO_ACTIVE_LOW;
                gpio_keys_100ask[i].irq  = gpio_to_irq(gpio_keys_100ask[i].gpio);
            }
            i-.0~count: request_irq(gpio_keys_100ask[i].irq, gpio_key_isr, IRQF_TRIGGER_RISING | IRQF_TRIGGER_FALLING, "100ask_gpio_key", &gpio_keys_100ask[i]);
            其中函数gpio_key_isr（）是执行的函数
        }
        static int gpio_key_remove(struct platform_device *pdev)
        {
            count = of_gpio_count(node);
            for (i = 0; i < count; i++){
                free_irq(gpio_keys_100ask[i].irq, &gpio_keys_100ask[i]);
            }
            kfree(gpio_keys_100ask);
        }
