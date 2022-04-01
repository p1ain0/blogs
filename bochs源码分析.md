# bochs源码分析

## 初始化过程分析

首先找到`main()`函数发现它也没干啥，把接受的命令行参数保存到`bx_startup_flags`全局变量中，然后普普通通的调了一个`bxmain`函数，`bxmain()`是执行初始化的主体，在这里面首先调用函数`bx_init_siminterface()`初始化模拟器接口，1.初始化全局变量`SIM`为`bx_real_sim_c`类，2.初始化`siminterface_log`类,作为日志输出的类，3.初始化`root_param`类。
回到`bxmain`函数里。然后使用`bx_init_main(bx_startup_flags.argc, bx_startup_flags.argv)`这个函数去执行其他的初始化任务。初始化`io`,`genlog`类

`bx_init_bx_dbg(void)`初始化bx_dbg结构体，这个结构体定义如下：

```c++
typedef struct {
  bool interrupts;
  bool exceptions;
  bool print_timestamps;
#if BX_DEBUGGER
  bool magic_break_enabled;
#endif
#if BX_GDBSTUB
  bool gdbstub_enabled;
#endif
#if BX_SUPPORT_APIC
  bool apic;
#endif
#if BX_DEBUG_LINUX
  bool linux_syscall;
#endif
} bx_debug_t;
```

`plugin_startup()`函数用来初始化插件入口点，插件有如下：

```c++
  pluginRegisterIRQ = builtinRegisterIRQ;
  pluginUnregisterIRQ = builtinUnregisterIRQ;

  pluginSetHRQHackCallback = builtinSetHRQHackCallback;
  pluginSetHRQ = builtinSetHRQ;

  pluginRegisterIOReadHandler = builtinRegisterIOReadHandler;
  pluginRegisterIOWriteHandler = builtinRegisterIOWriteHandler;

  pluginUnregisterIOReadHandler = builtinUnregisterIOReadHandler;
  pluginUnregisterIOWriteHandler = builtinUnregisterIOWriteHandler;

  pluginRegisterIOReadHandlerRange = builtinRegisterIOReadHandlerRange;
  pluginRegisterIOWriteHandlerRange = builtinRegisterIOWriteHandlerRange;

  pluginUnregisterIOReadHandlerRange = builtinUnregisterIOReadHandlerRange;
  pluginUnregisterIOWriteHandlerRange = builtinUnregisterIOWriteHandlerRange;

  pluginRegisterDefaultIOReadHandler = builtinRegisterDefaultIOReadHandler;
  pluginRegisterDefaultIOWriteHandler = builtinRegisterDefaultIOWriteHandler;
```

后边创建的`menu`变量，是存配置信息用的，这个数据结构设计的还是很值得我学习的，优雅简洁。之后有些特殊的配置信息，初始化到`root_param`链表里。



