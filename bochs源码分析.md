# bochsԴ�����

## ��ʼ�����̷���

�����ҵ�`main()`����������Ҳû��ɶ���ѽ��ܵ������в������浽`bx_startup_flags`ȫ�ֱ����У�Ȼ������ͨͨ�ĵ���һ��`bxmain`������`bxmain()`��ִ�г�ʼ�������壬�����������ȵ��ú���`bx_init_siminterface()`��ʼ��ģ�����ӿڣ�1.��ʼ��ȫ�ֱ���`SIM`Ϊ`bx_real_sim_c`�࣬2.��ʼ��`siminterface_log`��,��Ϊ��־������࣬3.��ʼ��`root_param`�ࡣ
�ص�`bxmain`�����Ȼ��ʹ��`bx_init_main(bx_startup_flags.argc, bx_startup_flags.argv)`�������ȥִ�������ĳ�ʼ�����񡣳�ʼ��`io`,`genlog`��

`bx_init_bx_dbg(void)`��ʼ��bx_dbg�ṹ�壬����ṹ�嶨�����£�

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

`plugin_startup()`����������ʼ�������ڵ㣬��������£�

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

��ߴ�����`menu`�������Ǵ�������Ϣ�õģ�������ݽṹ��ƵĻ��Ǻ�ֵ����ѧϰ�ģ����ż�ࡣ֮����Щ�����������Ϣ����ʼ����`root_param`�����



