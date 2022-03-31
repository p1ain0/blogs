# bochs源码分析

首先找到`main()`函数发现它也没干啥，把接受的命令行参数保存到`bx_startup_flags`全局变量中，然后普普通通的调了一个`bxmain`函数，`bxmain()`是执行初始化的主体，在这里面首先调用函数`bx_init_siminterface()`初始化模拟器接口，1.初始化全局变量`SIM`为`bx_real_sim_c`类，2.初始化`siminterface_log`类,作为日志输出的类，3.初始化`root_param`类。
回到`bxmain`函数里。然后使用`bx_init_main(bx_startup_flags.argc, bx_startup_flags.argv)`这个函数去执行其他的初始化任务。初始化`io`,`genlog`类