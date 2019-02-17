# Chapter 1
_______________________________________________________________________________________________________________
## 1.2.1 Oh, You Again...
```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

int main(void)
{
	int *ledInit = 0x24;
	int *ledData = 0x25;
	
	int *buttonInit = 0x21;
	int *buttonData = 0x22;
	int *buttonPin = 0x20;
	
	
	//int *buttonPin = 0x20;
	
	while (1)
	{
		*ledInit |= 0xff;
		*buttonInit |= 0b000000;
		*buttonData |= 0xff;
		
		while(1){
			_delay_ms(10);
			*ledData = *buttonPin;
		}
	}
}
```
_______________________________________________________________________________________________________________
## 1.2.2 Keep It Simple
```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

int main(void)
{
		DDRB = 0xff;
		DDRA = 0b000000;
		PORTA = 0xff;
		
		int prevPress = 0;
	
	while (1)
	{
		_delay_ms(100);
		if(PINA & (1<<PCINT4) && prevPress == 0){
			PORTB |= (1<<PCINT11);
			prevPress = 1;
		} else if(PINA & (1<<PCINT4) && prevPress == 1){
			PORTB = 0x00;
			prevPress = 0;
		}
	}
}
```
_______________________________________________________________________________________________________________
## 1.2.3 State Machine, once Moore
No it is not, the LED blinks when button B_Btn is pressed.
_______________________________________________________________________________________________________________
# Chapter 2
# 2.1
If the Baude rate changes the program does not work since the clocktime needs to be working with the AVR.
_______________________________________________________________________________________________________________
# 2.2
It works!
```
#define FOSC 1843200 // Clock Speed
#define BAUD 9600
#define MYUBRR FOSC/16/BAUD-1

#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

void usart0_init();
uint8_t usart0_receive();
void usart0_transmit(uint8_t data);
void init_LED();
void toggle_LED();

void main( void )
{
	usart0_init();
	init_LED();
	while(1){
		usart0_transmit(0x41);
		uint8_t receiveData = usart0_receive();
		usart0_transmit(receiveData);
		if(receiveData == 'A'){
			toggle_LED();
		}
	}
}

void usart0_init()
{
	/*Set baud rate */
	UBRR0H = (unsigned char)(103>>8);
	UBRR0L = (unsigned char)103;
	/*Enable receiver and transmitter */
	UCSR0B = (1<<RXEN0)|(1<<TXEN0);
	/* Set frame format: 8data, 2stop bit */
	UCSR0C |= (1<<UCSZ01)|(1<<UCSZ00);
	UCSR0C &= ~(1<<USBS0);
	}

uint8_t usart0_receive()
{
	/* Wait for data to be received */
	while ( !(UCSR0A & (1<<RXC0)) );
	/* Get and return received data from buffer */
	return UDR0;
}

void usart0_transmit(uint8_t data)
{
	/* Wait for empty transmit buffer */
	while ( !( UCSR0A & (1<<UDRE0)) );
	/* Put data into buffer, sends the data */
	UDR0 = data;
}

void init_LED(){
	DDRB = 0xff;
}

void toggle_LED(){

		PORTB |= (1<<PCINT11);
		_delay_ms(1000);
		PORTB = (0<<PCINT11);
	
}
```
_______________________________________________________________________________________________________________
# Chapter 3
## 3.1 - 3.4
The frequency is 16000000Hz/1024 = 15625Hz
T = 1/Hz
Duty cycle = PW/T * 100%
0.38% ~= 255 / 65535 * 100%

```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>
int main(void){
	
}

void timer3_init(){
	//Fast PWM with ICR3 as top
	TCCR3A = (1 << COM3A1)|(0 << COM3A0);
	TCCR3B = (1 << COM3B1)|(0 << COM3B0);

	//timer 3 with prescaler value 1024
	TCCR3B = (1 << CS32)|(0 << CS31)|(1 << CS30);

	//set PB6 as output
	PORTB = (1 << PCINT14);
	DDRB = (1 << DDB6);
}

void set_pulse(uint16_t val){
	OCR3A |= val;
}

void set_period(uint16_t val){
	ICR3 |= val;
}
```
_______________________________________________________________________________________________________________
## 3.5 
with period 0x130 the led apperars to be continuosly lit.
```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>
void timer3_init();
void set_pulse();
void set_period();

int main(void){
	timer3_init();
	set_pulse(0xff);
	set_period(0x130);
	while(1){
	}
}

void timer3_init(){
	//Fast PWM with ICR3 as top and set timer 3 with prescaler value 1024
	TCCR3A = (1 << COM3A1)|(0 << WGM31);
	TCCR3B = (1 << WGM33)|(0 << WGM32)|(1<<CS32)|(1<<CS30);

	//set PB6 as output
	DDRB = (1 << DDRB6);
}

void set_pulse(uint16_t val){
	OCR3A = val;
}

void set_period(uint16_t val){
	ICR3 = val;
}

```
_______________________________________________________________________________________________________________
## 3.6
Our guess is that it does not give us enought increments for controlling the period. Hencs we can not dim the lights without seeing the LED blinking.
_______________________________________________________________________________________________________________
## 3.7
304/255 number of effective pulses per period.
_______________________________________________________________________________________________________________
# Chapter 5
## 5.1
3590
```
/*
 * GccApplication1.c
 *
 * Created: 2019-02-11 08:22:20
 * Author : al3818ar-s
 */ 

#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

void timer3_init();
void set_pulse();
void set_period();

uint16_t dist;

int main(void){
	DDRC |= (1<<PORTC1);
	timer3_init();
	set_period(0xffff);
	while(1){
		PORTC |= (1<<PORTC1); //pin high
		_delay_ms(10);
		PORTC &= ~(1<<PORTC1); //pin low
		//Vänta på output US att bli high
		while((PINC & (1<<PORTC0)) == 0);
		//Starta timer
		TCCR1B |= (1<<CS10);
		TCNT1 = 0;
		//US output blir low igen
		while((PINC & (1<<PORTC0)) != 0);
		//Avsluta timer
		TCCR1B &= ~(1<<CS10);
		//Nu låt LED blicka beroende på distansen dist
		dist = TCNT1;
		set_pulse(dist);
		_delay_ms(60);
	}
}

void timer3_init(){
	//Fast PWM with ICR3 as top and set timer 3 with prescaler value 1024
	TCCR3A = (1 << COM3A1)|(0 << WGM31);
	TCCR3B = (1 << WGM33)|(0 << WGM32)|(1<<CS30);

	//set PB6 as output
	DDRB = (1 << PORTB6);
}

void set_pulse(uint16_t val){
	OCR3A = val;
}

void set_period(uint16_t val){
	ICR3 = val;
}
```
