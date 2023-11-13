# emWin窗口管理器

- 有效窗口在更改的时候已经完全更新，不需要重绘
- 无效窗口，不会反映所有更新，做出更改时，窗口管理器标记无效。下次窗口重绘时（手动或者回调函数）



### `WM_CALLBACK`

Pointer of type `WM_CALLBACK` which points to the callback function of the given window.

在`WM.h`文件中`typedef void WM_CALLBACK( WM_MESSAGE * pMsg);`

可以看的出`WM_MESSAGE`是一种函数指针，指代`void()(WM_MESSAGE * pMsg)`这种类型



## 消息机制

### `WM_PAINT`

 `WM_PAINT`是窗口变为无效，需要重绘时，发送到窗口。

 函数` GUI_Delay `非常关键，前面的章节中，我们多次强调它不仅仅是一个延迟函数，窗口管理器的执行就是通过它来实现的。

若要进入这个消息，需要这三个其中一个来触发：`GUI_Delay()`、`GUI_Exec()`、`WM_Exec()`

一般可以在这个消息里面实现自动重绘窗口



### `WM_CREATE`

窗口创建后立即发送，可以在此消息内初始化并创建任何子窗口.

`CreateWindows`注意和`WM_INIT_DIALOG`的区别



### `WM_INIT_DIALOG`

 `WM_INIT_DIALOG`    创建对话框后立即发送到对话框窗口。一般对话框上面的控件初始化，Windows或者FrameWin的初始化都是在这个消息里完成

`CreateDialogBox`注意和`WM_INIT_DIALOG`的区别

这个代码可以实现桌面窗口的重绘

```C
    // Set callback for background window
    //
    _cbOldBk = WM_SetCallback(WM_HBKWIN, _cbBkWindow);
    //
    // Move a window over background
    //
    _MoveWindow("Background has been redrawn");
    //
    // Delete callback for Background window
    //
    WM_SetCallback(WM_HBKWIN, _cbOldBk);
```



### `WM_NOTIFY_PARENT`

告诉父窗口，其子窗口中发生了某些改变。这个一般是GUI界面上的按钮按下释放



```C
switch(pMsg->Msg)
{
    case WM_NOTIFY_PARENT:
    	int Id = WM_GetId(pMsg->hWinSrc);
        Ncode = pMsg->Data.v;
        switch(Id)
        {
            case WM_CREATE:    
            	/* 设置聚焦 */
				WM_SetFocus(hWin);    
            break;
                
            case ID_BUTTON_0:   //按键
                switch(Ncode):
                {
                	case WM_NOTIFICATION_CLICKED:   //按钮已被点击
                    break;
                    
                    case WM_NOTIFICATION_RELEASED:  //按钮已被释放
                    break;
                    
                    case WM_NOTIFICATION_VALUE_CHANGED:  //按钮已被点击，且指针设备已移出按钮并且没有释放
                    break;                       
                }
            break;
              
           case ID_SCROLLBAR_0:   //滚动条
          	   switch(NCode)
              {
           		case WM_NOTIFICATION_CLICKED:
           		break;
                       
           		case WM_NOTIFICATION_RELEASED:
          		break;
                       
           		case WM_NOTIFICATION_VALUE_CHANGED:
           		break;
           break;    
           
           case ID_SLIDER_0:   //滑动块
               switch(NCode)
              {
           		case WM_NOTIFICATION_CLICKED:
           		break;
                       
           		case WM_NOTIFICATION_RELEASED:
          		break;
                       
           		case WM_NOTIFICATION_VALUE_CHANGED:
           		break;
              }
           break; 
                       
           case ID_CHECKBOX_0:    //复选框
               switch(NCode)
              {
           		case WM_NOTIFICATION_CLICKED:
           		break;
                       
           		case WM_NOTIFICATION_RELEASED:
          		break;
                       
           		case WM_NOTIFICATION_VALUE_CHANGED:
           		break;
              }
           break;             
      }
    break;
}
```



### `WM_KEY`

按下某个键后发送到当前**聚焦**的窗口。这个消息主要通过外部实体按键来发送。

`((WM_KEY_INFO*)(pMsg->Data.p))->Key 其中WM_MESSAGE *pMsg`, `pMsg`是`WM_MESSAGE`的结构体指针。Data.p指针指向WM_KEY_INFO结构体的消息

>```C
>struct WM_MESSAGE {
>  int MsgId;            /* type of message */
>  WM_HWIN hWin;         /* Destination window */
>  WM_HWIN hWinSrc;      /* Source window  */
>  union {
>    const void * p;     /* Some messages need more info ... Pointer is declared "const" because some systems (M16C) have 4 byte const, byte 2 byte default ptrs */
>    int v;
>    GUI_COLOR Color;
>  } Data;
>};
>```
>
>

```c
		 case WM_KEY:
			switch (((WM_KEY_INFO*)(pMsg->Data.p))->Key) 
            { 
				case GUI_KEY_ESCAPE:   //退出键
                    GUI_EndDialog(hWin, 1);
                    break;
				
				case GUI_KEY_TAB:     //指标键
					WM_SetFocusOnNextChild(hWin);
					break;
            }
         break;
```

实体按键通过`GUI_SendKeyMsg(int Key,int PressedCnt)`

> Key state. 1 for pressed state, 0 for released (unpressed) state.
>
> PressedCnt是按键的消息类型





### WM_TOUCH

指针输入设备被按下后，不管是移动，按下还是释放，都会发送此消息到窗口。





### `WM_TIMER`

`WM_CreateTimer()`创建向窗口发送` WM_TIMER` 消息的定时器

此函数用于创建定时器，注意，这个函数创建的定时器是单次的，也是说定时器时间到后定时器就不再工作了，如果还想继续使用，务必要在窗口回调函数的定时器消息`WM_TIMER` 里面调用函数`WM_RestartTimer` 重启此定时器。
