# XScuGic_Connect函数的用法
## 1.1定义
`XScuGic_Connect`用于将中断源（如外部硬件设备或内部模块）与特定的处理程序进行连接，以便在中断发生时调用该处理程序。其原函数为`s32  XScuGic_Connect(XScuGic *InstancePtr, u32 Int_Id, Xil_InterruptHandler Handler, void *CallBackRef){......}`
- `XScuGic *InstancePtr`指向`XScuGic`中断控制器实例的指针。它指向一个已初始化的中断控制器对象。
- `u32 Int_Id` 指定中断的 ID。每个中断源都有一个唯一的中断 ID。
- `Xil_InterruptHandler Handler`中断服务程序（ISR）的函数指针，这个函数将在中断触发时被调用。
- `void *CallBackRef`一个指针，可以传递给处理程序(`Handler`)的数据。这通常用于向中断处理程序传递额外的上下文信息，如硬件实例指针、状态信息等。
## 1.2实例

    //定义一个中断处理函数timer_intr_handler
    void timer_intr_handler(void *CallBackRef)
     {
      //LED状态,用于控制LED灯状态翻转
      static int led_state = 0;
      XScuTimer *timer_ptr = (XScuTimer *) CallBackRef;

      if(led_state == 0)
      {
          led_state = 1;
      }
      else
      {
          led_state = 0;
      }
      XGpioPs_WritePin(&Gpio, MIO_LED,led_state);
      XScuTimer_ClearInterruptStatus(timer_ptr);
     }
      
      //定时器中断初始化
   void timer_intr_init(XScuGic *intc_ptr,XScuTimer *timer_ptr)
    {
      //初始化中断控制器
      XScuGic_Config *intc_cfg_ptr;
      intc_cfg_ptr = XScuGic_LookupConfig(INTC_DEVICE_ID);
      XScuGic_CfgInitialize(intc_ptr, intc_cfg_ptr, intc_cfg_ptr->CpuBaseAddress);

      //设置并打开中断异常处理功能
      Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT, (Xil_ExceptionHandler)XScuGic_InterruptHandler, intc_ptr);
      Xil_ExceptionEnable();

      //设置定时器中断
      XScuGic_Connect(intc_ptr, TIMER_IRPT_INTR, (Xil_ExceptionHandler)timer_intr_handler, (void *)timer_ptr);

      XScuGic_Enable(intc_ptr, TIMER_IRPT_INTR); //使能GIC中的定时器中断
      XScuTimer_EnableInterrupt(timer_ptr);      //使能定时器中断
     }
      
在设置定时器中断处的代码`XScuGic_Connect(intc_ptr, TIMER_IRPT_INTR, (Xil_ExceptionHandler)timer_intr_handler, (void *)timer_ptr);`就将中断与自定义的中断程序关联了起来，每次中断触发时（此处为定时器中断`TIMER_IRPT_INTR`），就会执行中断程序。
