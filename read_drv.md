1. hello����: 
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
            copy_to_user(buf, kernel_buf, MIN(1024, size)); /* buf[]��user�ġ����ļ�����kernel_buf[1024] */
        }
        ssize_t hello_drv_write (struct file *file, const char __user *buf, size_t size, loff_t *offset){
            copy_from_user(kernel_buf, buf, MIN(1024, size)); /* buf[]��user�ġ����ļ�����kernel_buf[1024] */
        }
        int hello_drv_open (struct inode *node, struct file *file)
        int hello_drv_close (struct inode *node, struct file *file)


        struct class *hello_class;
        int __init hello_init(void){
            register_chrdev(0, "hello", &hello_drv);  /* ע��hello_drv�ṹ��Ϊ/dev/hello */
            class_create(THIS_MODULE, "hello_class"); /* ����class�ṹ�� */ �������õ���
            device_create(hello_class, NULL, MKDEV(major, 0), NULL, "hello"); /* �����豸�ڵ�MKDEV(���豸��, ���豸��)���֡�hello�� /dev/hello */
        }
        void __exit hello_exit(void){
            device_destroy(hello_class, MKDEV(major, 0));
            class_destroy(hello_class);
            unregister_chrdev(major, "hello");
        }


        module_init(hello_init); /* ���������ں���� */
        module_exit(hello_exit);

    ����:
        1.kconfig: �ṩ����menuconfig����
        2..config: ��������ֵ
        3.Makefile: ʹ������ֵ

2. �򵥵�led����:
    led_drv.c--main:


3. ������򵥵�board�ļ�:
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
                struct inode *inode = file_inode(file); /* ���ļ��л�ȡinode�ڵ� */
	            int minor = iminor(inode); /* �ӽṹ���л�ȡ���豸�� */

                copy_from_user(&status, buf, 1); /* ֻ��Ҫ0/1 */
                p_led_opr->ctl(minor, status); /* ����control����(���豸�ţ�״̬) */
        }
        int led_drv_open (struct inode *node, struct file *file){
            int minor = iminor(node);
            p_led_opr->init(minor); /* ��ʱ��ʼ���豸 */
        }
        int led_drv_close (struct inode *node, struct file *file)

        struct class *led_class;
        int __init led_init(void){
            register_chrdev(0, "100ask_led", &led_drv);  /* /dev/led */
            led_class = class_create(THIS_MODULE, "100ask_led_class");

            /* �õ���㲢�����ڵ㡢�������LED */
            p_led_opr = get_board_led_opr();
            for (i = 0; i < p_led_opr->num; i++)
		        device_create(led_class, NULL, MKDEV(major, i), NULL, "100ask_led%d", i); /* /dev/100ask_led0,1,... */
        }
        void __exit led_exit(void){
            /* ÿһ��������LED�����ٵ� */
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


4. ����������оƬ��Ӳ���豸:{
        board_A_led.c /* �����豸: ʹ��led_resource����GPIOx_y */
        chip_demo_gpio.c /* оƬ����: ʵ��led_operations������led_resource */
        led_drv.c /* �ϲ���������: ����led_operations���û�open��write��ϵinit��crl */
        led_resource.h /* ����'struct led_resource'�����豸���-x��y������ */
        led_opr.h /* ����'struct led_operations'���豸init��control */
}

    board_A_led.c--mian:

        struct led_resource board_A_led /* װ��ṹ�� */

        led_resource *get_led_resouce(void) /* ʵ�ֺ��� */


    chip_demo_gpio.c--main:

        struct led_resource *led_rsc

        struct led_operations board_demo_led_opr = {
        } /* ����led_operations�ṹ�� */

        int board_demo_led_init (int which){
            led_rsc = get_led_resouce(); /* ��board_A_led.c��ȡ��resource */
        }
        int board_demo_led_ctl (int which, char status)

        struct led_operations *get_board_led_opr(void) /* ʵ��get_board_led_opr */


    now.led_drv.c �� 3.led_drv.c һģһ��


5. bus���߰�: {
    board_A_led.c --> platform_device
    chip_demo_gpio.c --> platform_driver
    led_drv.c
}�ֱ���Ҫ�����.ko�ļ�������init������exit����

    led_opr.h 4.���� ʱһ��
        struct led_operations {
            int (*init) (int which);
            int (*ctl) (int which, char status);
        };

    
    led_resource.h:
        #define GROUP(x) (x>>16)
        #define PIN(x)   (x&0xFFFF)
        #define GROUP_PIN(g,p) ((g<<16) | (p))


    led_drv.h: /* ����led_drv.c��ĺ��� */
        void led_class_create_device(int minor);
        void led_class_destroy_device(int minor);
        void register_led_operations(struct led_operations *opr);


    board_A_led.c--main:

        /* platform_device�ṹ��ʵ�� */
        struct platform_device board_A_led_dev = {
            .name = "100ask_led",
            .num_resources = ARRAY_SIZE(resources),
            .resource = resources,
            .dev = {
                    .release = led_dev_release,
            },
        };
        ���е�resource:
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

        /* led_operations�ṹ��ʵ�� */
        struct led_operations board_demo_led_opr = {
            .init = board_demo_led_init,
            .ctl  = board_demo_led_ctl,
        };

        int board_demo_led_init (int which)       
        int board_demo_led_ctl (int which, char status)

        struct led_operations *get_board_led_opr(void)

        /* platform_driver�ṹ���ʵ�� */
        struct platform_driver chip_demo_gpio_driver = {
            .probe      = chip_demo_gpio_probe,
            .remove     = chip_demo_gpio_remove,
            .driver     = {
                .name   = "100ask_led", /* ƥ���name��־ */
            },
        };

        int g_ledpins[100]; 100��������
        int g_ledcnt = 0; ͳ���ж��ٸ���
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
             * ���Ծ���device�豸�ٿط�װ���������²�ʹ��
             * ��Ϊled_class��major�ڱ��㶨�壬�������豸���²�ָ��minor��led0/led1/...��
             */
            device_create(led_class, NULL, MKDEV(major, minor), NULL, "100ask_led%d", minor); /* /dev/100ask_led0,1,... */
        }
        void led_class_destroy_device(int minor){
            /* ��װ */
            device_destroy(led_class, MKDEV(major, minor));
        }
        void register_led_operations(struct led_operations *opr){
            /* ���²��*opr��������led_drv.c������� */
            p_led_opr = opr;
        }

        EXPORT_SYMBOL(led_class_create_device);
        EXPORT_SYMBOL(led_class_destroy_device);
        EXPORT_SYMBOL(register_led_operations);

        struct file_operations led_drv = { /* ���û����õ�file_operations�ӿں�֮ǰ����һ�� */
            .owner	 = THIS_MODULE,
            .open    = led_drv_open,
            .read    = led_drv_read,
            .write   = led_drv_write,
            .release = led_drv_close,
        };

        ...

        int __init led_init(void){
            /* ʡ����device_create()���裬�����²�(оƬ��chip)���� */
        }
        void __exit led_exit(void){
            /* ʡ��device_destroy() */
        }

        module_init(led_init);
        module_exit(led_exit);


6. �豸����: 
    �豸��: 
        dts(�豸��Դ�ļ�(dtsiΪ�豸��ͷ�ļ�)) --dtc--> dtb
        /des-v1/ -->�汾
        [memory reservations]  -->��ʽΪ/memreserve/ <address> <length>
        /{
            [label:] node-name[@unit-address]{
                [properties definition]
                [child nodes]
            }

            cpu{
                name = val -->��ʽ"string" <v32> <12>
                xxx
            };

            #address-cells = <1>; -->�������ݱ�ʾ��ַ
            #size-cells =<1>; -->�������ݱ�ʾ��С
            memory{
                reg = <0x80000000 0x20000000>
            }

            led{
                compatible = "A", "B", "C"; -->����
                model = "D"; -->����ABC��������D
                status = "ok" -->������"disable"
            }
        };

        �ڸ��ڵ��� ʹ��/�޸�:
            &lebal{
                xxx
            }
            &{/node-name@unit-address}{
                xxx
            }


7. ��������: (����Ҫֱ�Ӳ����Ĵ���)
    pinctrl:
        1.���豸�����涨��һ��UART�ڵ�(ʹ��pinctrl����Ϊclient_device)
            UART{
                /* state: xxx -- group: yyy1, yyy2 -- function: zzz */
                pinctrl-names = "defualt", "sleep";
                pinctrl-0 = <&xxx_0>;
                pinctrl-1 = <&xxx_1>; /* xxx-0 �� xxx_1 λ��pinctrl���� */
                status = "okay";
            }
        2.����һ��pincontroller�ڵ�:
            pincontroller{
                xxx_0{ /* ���ýڵ� */
                    function = "uart";
                    groups = "", "";
                } /* Ĭ��״̬ʱuart */

                xxx_1{ /* ���ýڵ� */
                    groups = "", "";
                    output-high��
                } /* sleepʱ����Ϊ�ߵ�ƽ */
            }

    GPIO: (gpiod: new, gpio: old)
        1.���豸�����洴��device�ڵ�:
            device{
                led-gpio = <A, xx, flag>;
            }
        2.�豸��������һ���ڵ�(��������gpio-controller):
            gpioA: gpio@xxx{
                gpio-controller;
                #gpio-cells = <2>;
            }

    INT:
        1.�豸��������Ҫ���жϵ��豸:
            device{
                interrupt-parent = <&gpioA>;
                interrupt = <xx, type>; /* xx(hwirq)����domain*/
            }
        2.ʹ��irq_domain(hwirq->soft_irq)
            struct irq_domain{
                struct irq_domain_ops{
                        .map
                        .xlate -->�����õ�irq(�����жϺ�)-irq_request
                        ......
                    }
                int linear_revmap[] -->(hwirq, irq)
                ......
            }
        3.�豸��������һ���ڵ�(��������interrupt-controller)
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
                interrupts = <>, <>, <>; /* ʹ��parent����һ�������жϣ�����ÿһ��<>����ж���parent->cells = <?>���� */
            }

    �ж�GPIO:
        1.��ԭ��ͼ --> ȷ���ĸ�GPIO��Ϊ�����ж�
        2.���豸����ӽڵ� --> xxx_gpio_key{
                                    compatible = "xxx_gpio_key";
                                    gpios = <&gpioA x xxxx
                                                &gpioB y yyyy>;
                                    pinctrl-names = "default";
                                    pinctrl-0 = <&keyA_pinctrl
                                                    &keyB_pinctrl>;
                                }
                            /* ����ָ���ж�: gpiod_to_irq()ת�����ж� */
        3.����platform_driver, ��.pr obe�����ȡgpio, ת����irq
        4.request_irq

8. �����豸gpio�жϳ���:
    1.����platform_driver
        static const struct of_device_id ask100_keys[] = {
            { .compatible = "100ask,gpio_key" },
            { },
        };
        static struct platform_driver gpio_keys_driver = {
            .probe      = gpio_key_probe,
            .remove     = gpio_key_remove,
            .driver     = {
                .name   = "100ask_gpio_key",
                .of_match_table = ask100_keys, //�豸�ڵ��compatible
            },
        };
    
    2.����ں���:
        static int __init gpio_key_init(void)
        static void __exit gpio_key_exit(void)

    3.ʵ��gpio_key_probe��gpio_key_remove
        struct gpio_key{
            int gpio;
            struct gpio_desc *gpiod;
            int flag;
            int irq;
        };
        static struct gpio_key *gpio_keys_100ask;
        
        int gpio_key_probe(struct platform_device *pdev){
            count = of_gpio_count(node); //ȷ������gpio
            gpio_keys_100ask = kzalloc(sizeof(struct gpio_key) * count, GFP_KERNEL);
            
            for (i = 0; i < count; i++){
                gpio_keys_100ask[i].gpio = of_get_gpio_flags(node, i, &flag); //��ȡ��i��gpio������flag
                /* ���������� */
                gpio_keys_100ask[i].gpiod = gpio_to_desc(gpio_keys_100ask[i].gpio);
                gpio_keys_100ask[i].flag = flag & OF_GPIO_ACTIVE_LOW;
                gpio_keys_100ask[i].irq  = gpio_to_irq(gpio_keys_100ask[i].gpio);
            }
            i-.0~count: request_irq(gpio_keys_100ask[i].irq, gpio_key_isr, IRQF_TRIGGER_RISING | IRQF_TRIGGER_FALLING, "100ask_gpio_key", &gpio_keys_100ask[i]);
            ���к���gpio_key_isr������ִ�еĺ���
        }
        static int gpio_key_remove(struct platform_device *pdev)
        {
            count = of_gpio_count(node);
            for (i = 0; i < count; i++){
                free_irq(gpio_keys_100ask[i].irq, &gpio_keys_100ask[i]);
            }
            kfree(gpio_keys_100ask);
        }
