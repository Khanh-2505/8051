# 8051
#include <REGX52.H>

#define in P3_2			// chan do toc do
#define LED1 P2_0
#define LED2 P2_1
#define LED3 P2_2	// 
#define LED4 P2_3 // Dat ten chan Led so 4
// Bien do toc do
unsigned int Count = 0, n = 0;				// So xung dem vao cua ENCODER
unsigned int Time = 0;					// Dem 1000 lan 1ms -> 1s 
unsigned int x = 0, a = 0, b = 0, Van_Toc = 0;
unsigned char LED7SEGCA[10] = {0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8 , 0x80, 0x90};

unsigned char Duty;		// Phan tram do rong xung 
unsigned char Count_Duty; // Bien dem gia tri xungc
unsigned int ck, Ton, Toff, T = 1000;
unsigned char Ton_h_reload, Ton_l_reload, Toff_h_reload, Toff_l_reload;

sbit Pwm_Pin = P2^4;    // Chan tao xung dieu khien L293D
sbit Pwm_B	= P2^5;		// Nut nhan dieu chinh che do 1
//sbit Pwm_B_60 = P2^6;
//sbit Pwm_B_90 = P2^7;
// Kiem tra ngat ngoai
void NgatINT0(void) interrupt 0 using 1
{
	Count++;
}
void timer1() interrupt 3 using  2
{
	Time++;
	TH1 = 0xFC;			// khoi tao ngat 1ms
	TL1 = 0x18;
	TR1 = 1;
	if(Time == 1000) // ngat 1s
	{
		x = Count;
		Time = 0;
		Count = 0;
		a = (int)x * 0.6;
		Van_Toc  = (int)(a + b) / 2;
		b = a;
	}
}
// Ham khoi tao timer 1
void init()
{
	TMOD = 0x10;
	in =1;
	
	EX0=1; // Cho phep dung ngat ngoai 0
	IT0=1; // Kiem tra dieu kien ngat ngoai tai INT0
	
	ET1=1; // Cho phep dung timer 1
	TR1=1; // Bat timer
	EA = 1; // cho phep ngat toan cuc
}
// Ham tao tre
void Delay_ms(unsigned int t)
{
	unsigned int i, j;
	for(i = 0; i < t; i++)
	{
		for( j = 0; j < 123; j++)
		{
			// Nothing
		}
	}	
}
 //Ham hien thi 4 LED 7 thanh
void Hien_thi(unsigned int Van_Toc)
{
		//STemp = Van_Toc;
		// So nghin
		P0 = LED7SEGCA[Van_Toc/1000];
		LED1 = 0; 
		Delay_ms(5);
		LED1 = 1;
		
		// So hang tram
		P0 = LED7SEGCA[Van_Toc/100];
		LED2 = 0;
		Delay_ms(5);
		LED2 = 1;
		
		// So hang chuc
		P0 = LED7SEGCA[(Van_Toc%100)/10];
		LED3 = 0;
		Delay_ms(5);
		LED3 = 1;
		
		// So hang don vi
		P0 = LED7SEGCA[Van_Toc%10];
		LED4 = 0;
		Delay_ms(10);
		LED4 = 1;
}
// Ham khoi tao timer 0
void Pwm_Init()
{
	Pwm_Pin = 0;
	
	TMOD &= 0xF0;
	TMOD |= 0x01;
	TH0 = 0xFF;
	TL0 = 0x9C;
	ET0 = 1;
	TR0 = 1;
}
//Ham ngat timer 0 ngat xung 100us
void Timer0overflow() interrupt 1
{
	Count_Duty++;
	if(Count_Duty < Duty)
	{
		Pwm_Pin = 1;
		TH0 = 0xFF;
		TL0 = 0x9C;
		TR0 = 1;
	}
	else 
	{
		TH0 = 0xFF;
		TL0 = 0x9C;
		TR0 = 1;
		Pwm_Pin = 0;
	}
	if(Count_Duty == 100)
	{
		Count_Duty = 0;
	}
}
// Ham kiem tra nut nhan
void Button()
{
	if(Pwm_B == 0)
	{
		while(!Pwm_B) {;}
		n++;
	}
	if(n == 4)
	{
		n = 0;
	}
	switch (n)
	{
		case 1:
			Duty = 30;
			break;
		case 2: 
			Duty = 60;
			break;
		case 3:
			Duty = 90;
			break;
	}
}
// Ham chinh 
void main(void)
{
	init();
	Pwm_Init();
	while(1)
	{	
		Button();
		Hien_thi(Van_Toc);
	}
}
