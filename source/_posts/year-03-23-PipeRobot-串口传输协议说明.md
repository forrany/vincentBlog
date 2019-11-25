---
title: PipeRobot 串口传输协议说明
date: 2019-03-23 15:42:38
tags: 
    - 串口协议说明
---

树莓派作为服务器，接收手机/PC端发送的控制指令后，通过UART串口将控制指令进行转发，这里对指令协议进行说明

## 通讯协议

| 数据编号 | 数据内容 | 含义     |
| :------- | -------: | :------: |
| 0        | 0x55     | 包头     |
| 1        | command  | 控制指令 |
| 2        | speed    | 速度     |

每帧数据都包含了3个字节的数据，分别为包头、控制指令和速度，其中包头均以`0x55`开头，可以在接收数据时做简单的校验。

### 控制指令

第二个字节控制指令主要包括了：
* 前进 `0x01`
* 后退 `0x02`
* 伸张 `0x03`
* 收缩 `0x04`
* 停止 `0x05`

下位机在接受数据时，根据第二字节数据判断指令内容

### 速度

第三个字节是速度，范围是`[0x00,0x64]`(即十进制的0~100)。

下位机在接收到数据时，可以根据速度对电机的转速进行相应的控制

## 下位机程序参考

下位机使用ARM系列芯片，实现对机器人的控制，这里主要对串口协议部分给出程序参考

### 中断部分

```C
unsigned char Re_buf[11],counter=0;
unsigned char sign;
void USART2_IRQHandler(void)		   //串口2全局中断服务函数
{
	if(USART_GetITStatus(USART2, USART_IT_RXNE) != RESET)  //接收中断有效,若接收数据寄存器满
  	{
		Re_buf[counter] = USART_ReceiveData(USART2);	//接收数据
		if(counter == 0 && Re_buf[0] != 0x55) return;      //第 0 号数据不是帧头，跳过
		counter++; 
		if(counter==3) //接收到 11 个数据
		{ 
			counter=0; //重新赋值，准备下一帧数据的接收
			sign=1;  //sign作为标志，表示收到数据，可以在主程序中检测该状态，以进行相应的内容
		}
	}
}

```

### 主程序

在主程序中，根据指令的内容，进行不同的控制
```C
u16 speed;
extern unsigned char Re_buf[11],counter;
extern unsigned char sign;
 while (1)
   {
      if(sign)
      {  
        sign = 0;
        if(Re_buf[0]==0x55)       //检查帧头
        {  
           speed = speedMap(Re_buf[2]); //写一个函数，将0~100映射到0~999
           switch(Re_buf[1])
           {
              case 0x01: //标识前进
                Motor1_control(0,speed) //前进
                break;
              case 0x02: //标识后退
                Motor1_control(speed,0) //前进
                break;
              case 0x53: //标识伸长
                Motor2_control(900,0); //直接900的速度伸长，忽略速度
                break;
              case 0x54: //标识收缩
                Motor2_control(0,900);直接900的速度收缩，忽略速度
                break;
              case 0x55: //标识停止
                GPIO_ResetBits(GPIOC,GPIO_Pin_7); //拉低使能端
                break;
              default:  break;
           }			
        }   
      }
```

#### 注意点

收缩和伸展均以较高的速度进行，原因在于原程序中使用了编码器检测速度变化的方式，来自动停止伸缩和伸长。

当速度较低的时，会影响其正常工作，且一般伸长、收缩需要尽快完成。