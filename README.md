#include <reg52.h>
#include <stdio.h>

// 定义引脚
sbit buzzer = P1^0;
sbit stepMotorA = P1^1;
sbit stepMotorB = P1^2;
sbit stepMotorC = P1^3;
sbit stepMotorD = P1^4;
sbit LCD_RS = P2^0;
sbit LCD_RW = P2^1;
sbit LCD_EN = P2^2;
// 矩阵按键引脚定义
sbit KEY_ROW1 = P3^0;
sbit KEY_ROW2 = P3^1;
sbit KEY_ROW3 = P3^2;
sbit KEY_ROW4 = P3^3;
sbit KEY_COL1 = P3^4;
sbit KEY_COL2 = P3^5;
sbit KEY_COL3 = P3^6;
sbit KEY_COL4 = P3^7;

// 函数声明
void delay(unsigned int ms);
void LCD_Init(void);
void LCD_WriteCmd(unsigned char cmd);
void LCD_WriteData(unsigned char dat);
void LCD_DisplayString(unsigned char x, unsigned char y, char *str);
void LCD_DisplayNum(unsigned char x, unsigned char y, unsigned int num);
void keyScan(void);
void floorControl(void);
void buzzerControl(void);
void stepMotorMove(bit direction, unsigned char steps);

unsigned char currentFloor = 1;
unsigned char targetFloor = 1;
bit isUp = 0;
bit doorStatus = 0;
unsigned char keyValue = 0;

// 延时函数
void delay(unsigned int ms)
{
    unsigned int i, j;
    for(i = ms; i > 0; i--)
        for(j = 110; j > 0; j--);
}

// LCD 初始化函数
void LCD_Init(void)
{
    LCD_WriteCmd(0x38); // 8 位数据接口，2 行显示，5x7 点阵
    LCD_WriteCmd(0x0C); // 显示开，光标关，闪烁关
    LCD_WriteCmd(0x06); // 光标移动方向，光标自动加 1
    LCD_WriteCmd(0x01); // 清屏
}

// 向 LCD 写入命令函数
void LCD_WriteCmd(unsigned char cmd)
{
    LCD_RS = 0;
    LCD_RW = 0;
    P0 = cmd;
    delay(5);
    LCD_EN = 1;
    delay(5);
    LCD_EN = 0;
}

// 向 LCD 写入数据函数
void LCD_WriteData(unsigned char dat)
{
    LCD_RS = 1;
    LCD_RW = 0;
    P0 = dat;
    delay(5);
    LCD_EN = 1;
    delay(5);
    LCD_EN = 0;
}

// 在 LCD 上指定位置显示字符串函数
void LCD_DisplayString(unsigned char x, unsigned char y, char *str)
{
    unsigned char i = 0;
    if(y == 0)
        LCD_WriteCmd(0x80 + x);
    else
        LCD_WriteCmd(0xC0 + x);
    while(str[i] != '\0')
    {
        LCD_WriteData(str[i]);
        i++;
    }
}

// 在 LCD 上指定位置显示数字函数
void LCD_DisplayNum(unsigned char x, unsigned char y, unsigned int num)
{
    char str[5];
    unsigned char i;  // 将循环变量移到函数开头声明
    
    sprintf(str, "%d", num);
    if(y == 0)
        LCD_WriteCmd(0x80 + x);
    else
        LCD_WriteCmd(0xC0 + x);
    
    for(i = 0; str[i] != '\0'; i++)
    {
        LCD_WriteData(str[i]);
    }
}

// 矩阵按键扫描函数
void keyScan(void)
{
    KEY_ROW1 = 0; KEY_ROW2 = 1; KEY_ROW3 = 1; KEY_ROW4 = 1;
    if(KEY_COL1 == 0) { delay(20); if(KEY_COL1 == 0) keyValue = 1; while(KEY_COL1 == 0); }
    if(KEY_COL2 == 0) { delay(20); if(KEY_COL2 == 0) keyValue = 2; while(KEY_COL2 == 0); }
    if(KEY_COL3 == 0) { delay(20); if(KEY_COL3 == 0) keyValue = 3; while(KEY_COL3 == 0); }
    if(KEY_COL4 == 0) { delay(20); if(KEY_COL4 == 0) keyValue = 4; while(KEY_COL4 == 0); }

    KEY_ROW1 = 1; KEY_ROW2 = 0; KEY_ROW3 = 1; KEY_ROW4 = 1;
    if(KEY_COL1 == 0) { delay(20); if(KEY_COL1 == 0) keyValue = 5; while(KEY_COL1 == 0); }
    if(KEY_COL2 == 0) { delay(20); if(KEY_COL2 == 0) keyValue = 6; while(KEY_COL2 == 0); }
    if(KEY_COL3 == 0) { delay(20); if(KEY_COL3 == 0) keyValue = 7; while(KEY_COL3 == 0); }
    if(KEY_COL4 == 0) { delay(20); if(KEY_COL4 == 0) keyValue = 8; while(KEY_COL4 == 0); }

    KEY_ROW1 = 1; KEY_ROW2 = 1; KEY_ROW3 = 0; KEY_ROW4 = 1;
    if(KEY_COL1 == 0) { delay(20); if(KEY_COL1 == 0) keyValue = 9; while(KEY_COL1 == 0); }
    if(KEY_COL2 == 0) { delay(20); if(KEY_COL2 == 0) keyValue = 10; while(KEY_COL2 == 0); }
    if(KEY_COL3 == 0) { delay(20); if(KEY_COL3 == 0) keyValue = 0; while(KEY_COL3 == 0); }
    if(KEY_COL4 == 0) { delay(20); if(KEY_COL4 == 0) keyValue = 0; while(KEY_COL4 == 0); }

    KEY_ROW1 = 1; KEY_ROW2 = 1; KEY_ROW3 = 1; KEY_ROW4 = 0;
    if(KEY_COL1 == 0) { delay(20); if(KEY_COL1 == 0) keyValue = 0; while(KEY_COL1 == 0); }
    if(KEY_COL2 == 0) { delay(20); if(KEY_COL2 == 0) keyValue = 0; while(KEY_COL2 == 0); }
    if(KEY_COL3 == 0) { delay(20); if(KEY_COL3 == 0) keyValue = 0; while(KEY_COL3 == 0); }
    if(KEY_COL4 == 0) { delay(20); if(KEY_COL4 == 0) keyValue = 0; while(KEY_COL4 == 0); }
}

// 步进电机控制函数
void stepMotorMove(bit direction, unsigned char steps)
{
    unsigned char i;
    if(direction) // Upward
    {
        for(i = 0; i < steps; i++)
        {
            stepMotorA = 1; stepMotorB = 0; stepMotorC = 0; stepMotorD = 0; delay(5);
            stepMotorA = 0; stepMotorB = 1; stepMotorC = 0; stepMotorD = 0; delay(5);
            stepMotorA = 0; stepMotorB = 0; stepMotorC = 1; stepMotorD = 0; delay(5);
            stepMotorA = 0; stepMotorB = 0; stepMotorC = 0; stepMotorD = 1; delay(5);
        }
    }
    else // Downward
    {
        for(i = 0; i < steps; i++)
        {
            stepMotorA = 0; stepMotorB = 0; stepMotorC = 0; stepMotorD = 1; delay(5);
            stepMotorA = 0; stepMotorB = 0; stepMotorC = 1; stepMotorD = 0; delay(5);
            stepMotorA = 0; stepMotorB = 1; stepMotorC = 0; stepMotorD = 0; delay(5);
            stepMotorA = 1; stepMotorB = 0; stepMotorC = 0; stepMotorD = 0; delay(5);
        }
    }
}

// 楼层控制函数
void floorControl(void)
{
    if(keyValue > 0 && keyValue <= 8)
    {
        targetFloor = keyValue;
        if(targetFloor > currentFloor)
        {
            isUp = 1;
            stepMotorMove(1, (targetFloor - currentFloor) * 50); // 每层50步
            currentFloor = targetFloor;
        }
        else if(targetFloor < currentFloor)
        {
            isUp = 0;
            stepMotorMove(0, (currentFloor - targetFloor) * 50); // 每层50步
            currentFloor = targetFloor;
        }
    }
    else if(keyValue == 10) // Door open/close button
    {
        doorStatus = ~doorStatus;
    }
    keyValue = 0;
}

// 蜂鸣器控制函数
void buzzerControl(void)
{
    if(currentFloor == targetFloor)
    {
        buzzer = 1;
        delay(500);
        buzzer = 0;
    }
}

void main()
{
    LCD_Init();
    LCD_DisplayString(0, 0, "Floor:    Dir:");
    LCD_DisplayString(0, 1, "Target:   Door:");
    
    while(1)
    {
        keyScan();
        floorControl();
        buzzerControl();

        // 显示当前楼层
        LCD_DisplayNum(6, 0, currentFloor);
        
        // 显示方向状态
        if(isUp)
            LCD_DisplayString(14, 0, "UP ");
        else
            LCD_DisplayString(14, 0, "DO");

        // 显示目标楼层
        LCD_DisplayNum(7, 1, targetFloor);

        // 显示门状态
        if(doorStatus)
            LCD_DisplayString(14, 1, "OP ");
        else
            LCD_DisplayString(14, 1, "CL");

        delay(100);
    }
}
