# Chapter 1
________________________________________________________________________________________________________________________________________
## 1.1
Is it necessary to wait for the whole sequence that is generated when a lion passes through the
hallway?

  No it is not since we store halfwaypoints in variables.
________________________________________________________________________________________________________________________________________
## 1.2
How fast can an obstacle move through the sensor array without it being missed?

  Quite fast, we have the clock frequency 16 MHz and the process takes a number of clock cycles witch is the bottle neck.
________________________________________________________________________________________________________________________________________
## 1.2.2 high-tech Lion cage
```
#define F_CPU 16000000UL
#include <avr/io.h>

#define S1 PORTC6
#define S2 PORTC7

uint8_t lions = 0;
int goesOut = 0;
int middle = 0;
int goesIn = 0;


int main(void)
{
	sensor_init();
	//led_init();
	security_system_init();
	while (1){
		security_system_run();
		send_lions(lions);
		//innan något har hänt, vilket håll går lejonet
		if(read_s1() && (!goesOut || !goesIn)){
			goesOut = 1;
		}
		if(read_s2() && (!goesOut || !goesIn)){
			goesIn = 1;
		}
		
		if(goesOut){
			if((read_s1() && read_s2()) || middle){
				middle = 1;
				if(!read_s2()){
					lions++;
					//led_toggle(lions);
					middle = 0;
					goesOut = 0;
				}
			}
		}

		if(goesIn){
			if((read_s1() && read_s2()) || middle){
				middle = 1;
				if(!read_s1()){
					lions--;
					//led_toggle(lions);
					middle = 0;
					goesIn = 0;
				}
			}
		}
		
		
	}
}

//Dessa tre bör vara rätt
void sensor_init(){
	DDRC |= (0<<DDRC6) | (0<<DDRC7);
}

int read_s1(){
	return PINC & (1<<S1);
}
```
___________________________________________________________________________________________________________________________________
## 1.4
If a lion moves fast through the senors is it still registered? If not, why? Isn’t that a strange(r)
thing(s)? Are there any limitations to consider while polling sensors?

	No it does not seem to work, that is truly peculiar. One might say, it is indeed a strange(r) thing(s). 
	Our solution would be to sampe more often. Isn't it a crazy fast moving wrld we live in?

___________________________________________________________________________________________________________________________________
# Chapter 2
__________________________________________________________________________________________________________________________________
## 2.1
What is the value of the variable?
	
	It is now 4, due to 4 states of the sensor.

__________________________________________________________________________________________________________________________________
## 2.2.2 Resolve the issue, part 2
```
#include <avr/io.h>
#include <avr/interrupt.h>
#define interuptPins PCMSK2
#define S1Interupt PCINT23
#define S2Interupt PCINT22
#define S1 PORTC6
#define S2 PORTC7

volatile uint8_t lions;
volatile int goesOut = 0;
volatile int middle = 0;
volatile int goesIn = 0;

int main(void)
{
	//init
	interupt_init();
	sei();
	security_system_init();
	
	while (1)
	{
		// run security
		security_system_run();
	}
}

void interupt_init(){
	PCICR |= (1<<PCIE2); // Since we have PCINT22/23, we use PCIE2 in PCICR to enable Pin change interrupt
	interuptPins |= (1<<S1Interupt)|(1<<S2Interupt); // Enable the pins
}


int read_s1(){
	return PINC & (1<<S1);
}
int read_s2(){
	return PINC & (1<<S2);
}


ISR(PCINT2_vect){
	//innan något har hänt, vilket håll går lejonet
	if(read_s1() && (!goesOut || !goesIn)){
		goesOut = 1;
	}
	if(read_s2() && (!goesOut || !goesIn)){
		goesIn = 1;
	}

	if(goesOut){
		if((read_s1() && read_s2()) || middle){
			middle = 1;
			if(!read_s2()){
				lions++;
				middle = 0;
				goesOut = 0;
			}
		}
	}

	if(goesIn){
		if((read_s1() && read_s2()) || middle){
			middle = 1;
			if(!read_s1()){
				lions--;
				middle = 0;
				goesIn = 0;
			}
		}
	}
	send_lions(lions);
}
```
__________________________________________________________________________________________________________________________________
## 2.2
Are there any disadvantages to consider while using interrupts? Can it be known which part
of the code that is executed just before an interrupt?

	Yes there are disadvantages. Since the interrupt jumps out of the code block the instance it gets interrupted some code might 		not be executed right. This example shows the problem.
	
```
var x;
x = x^2 * sin(x) + x;

ISR(){
	x = 0;
}
```
	If the code gets interrupted while calculating x we get the value 0 which might not be what we are looking for.
__________________________________________________________________________________________________________________________________
## 2.2.3
__________________________________________________________________________________________________________________________________
## 2.3
What is the value of the variable? What did you send from the terminal? Is it correct? (Hint:
An ASCII table might help.)
	We send b and recieve the value 98, which is correct.
__________________________________________________________________________________________________________________________________
## 2.4
Do you observe anything strange regarding the Neo-pixels while transmitting the string?
	It blinks blue or green sometimes.
__________________________________________________________________________________________________________________________________
## 2.5
What is causing the Neo-pixels to randomly light up? (Hint: When does the interrupt occur?
Is it possible to know what part of the code that is executed when the interrupt occurs?)
	The interrupt may cause the send_one or send_zero function to be interrupted.
__________________________________________________________________________________________________________________________________
## 2.6
How could you solve the problem?
	Using cli() to not use global interrupts
__________________________________________________________________________________________________________________________________
What does the cli() function do?
	
__________________________________________________________________________________________________________________________________
## 2.7
At what cost is the problem solved?


```
#define F_CPU 16000000UL

#include <avr/io.h>
#include <util/delay.h>
#include <string.h>
#include <avr/interrupt.h>

#define DATA_PORT PORTD
#define DATA_PIN 5
#define DATA_DDR DDRD

#define TRIG	(PC1)
#define ECHO	(PC0)

#define NBR_PIXELS (10)
#define SPEED (75)

#define BTN1	0x1
#define BTN2	0x2
#define BTN3	0x4
#define BTN4	0x8
#define BTN5	0x10
#define BTN6	0x20

enum direction {LEFT = 0, RIGHT = 1, PAUSE = 2};

typedef struct {
	uint8_t r;
	uint8_t g;
	uint8_t b;
} pixel_t;

pixel_t leds[10];
pixel_t leds_tmp[10];

void send_one();
void send_zero();

// ----- Global variable, to be set in assembly code, by YOU. -----
char button_state;
volatile uint8_t recieveData;

// Implemented to you by the authors :)
void init();
void fill_array(uint8_t *, uint8_t, uint8_t);
void clear_leds();
void set_pixels(pixel_t *);
uint16_t us_measure();
void loop();
uint16_t delay_loop = 30;

void usart0_init();

// ###################################################### Add your UART initialization code prototype here!

int main(void)
{
	usart0_init();
	sei();
	init();
	
	
	// init state
	leds[1].r = 25;
	leds[2].r = 50;
	leds[3].r = 100;
	leds[4].r = 50;
	leds[5].r = 25;
	
	sei();
	
	while (1) {

		loop();
		
	}
}

// ###################################################### Add your UART initialization code here!
void usart0_init()
{
	UCSR0B |= (1 << RXEN0) | (1 << TXEN0);
	UCSR0C |= (1 << UCSZ10)|(1 << UCSZ11);
	UBRR0 = 103; //9600
	UCSR0B |= (1 << 7);
}

// ###################################################### Add your ISR here!

void init() {
	// neopixels
	DATA_DDR = (1 << DATA_PIN);
	DATA_PORT &= ~(1 << DATA_PIN);


}


void loop() {
	
	uint8_t state = 1;
	uint8_t dir = LEFT;
	
	
	
	switch (PINA >> 2) {
		case BTN1:
		// set the color of the LEDs to red
		clear_leds();
		leds[state].r = 25;
		leds[(state + 1) % 10].r = 50;
		leds[(state + 2) % 10].r = 100;
		leds[(state + 3) % 10].r = 50;
		leds[(state + 4) % 10].r = 25;
		break;
		case BTN2:
		// set the color of the LEDs to green
		clear_leds();
		leds[state].g = 25;
		leds[(state + 1) % 10].g = 50;
		leds[(state + 2) % 10].g = 100;
		leds[(state + 3) % 10].g = 50;
		leds[(state + 4) % 10].g = 25;
		break;
		case BTN3:
		// set the color of the LEDs to blue
		clear_leds();
		leds[state].b = 25;
		leds[(state + 1) % 10].b = 50;
		leds[(state + 2) % 10].b = 100;
		leds[(state + 3) % 10].b = 50;
		leds[(state + 4) % 10].b = 25;
		break;
		case BTN4:
		dir = RIGHT;
		break;
		case BTN5:
		//dir = PAUSE;
		
		delay_loop = us_measure();
		
		break;
		case BTN6:
		dir = LEFT;
		break;
	}

	if (dir == LEFT) {
		for (uint8_t i = 9; i > 0; i--) {
			leds_tmp[i] = leds[i-1];
		}
		leds_tmp[0] = leds[9];
		} else if (dir == RIGHT) {
		leds_tmp[9] = leds[0];
		for (uint8_t i = 0; i < 9; i++) {
			leds_tmp[i] = leds[i+1];
		}
		} else if (dir == PAUSE) {
		for (int i = 0; i < 10; i++) {
			leds_tmp[i] = leds[i];
		}
	}

	set_pixels(leds_tmp);
	if (dir == LEFT) {
		state = (state + 1) % 10;
		} else if (dir == RIGHT) {
		state = (((state - 1) % 10) + 10) % 10;
		} else if (dir == PAUSE) {
		// pause
	}
	
	memcpy(leds, leds_tmp, 240);

	_delay_us(50);

	for (uint8_t i = 0; i < delay_loop; i++) {
		_delay_ms(2);
	}
	
}


void fill_array(uint8_t* arr, uint8_t color, uint8_t off) {
	for(int i = 0; i < 8; i++) {
		// expand a byte into 8 uint8_t binary values in array for faster access later
		arr[i + off] = (color & (1 << (7 - i))) >> (7-i);
	}
}

void clear_leds() {
	for (int i = 0; i < 10; i++) {
		leds[i].r = 0;
		leds[i].g = 0;
		leds[i].b = 0;
	}
	set_pixels(leds);
}


void set_pixels(pixel_t *p) {
	uint8_t led_expand[24 * NBR_PIXELS];
	for (uint8_t i = 0; i < 10; i++) {
		// retarded ordering...
		fill_array(led_expand, p[i].g, i * 24 + 0);
		fill_array(led_expand, p[i].r, i * 24 + 8);
		fill_array(led_expand, p[i].b, i * 24 + 16);
	}
	
	// send values to neopixel
	for (uint8_t i = 0; i < 24 * NBR_PIXELS; i++) {
		if (led_expand[i]) {
			// send 1
			send_one();		//<=============================== Sending pixel data
			} else {
			// send 0
			send_zero();	//<=============================== Sending pixel data
		}
	}
}



uint16_t us_measure() {
	
	DDRC |= _BV(TRIG);
	TCNT1 = 0;
	PORTC |= _BV(TRIG);
	_delay_us(10);
	PORTC &= ~_BV(TRIG);
	
	while(!(PINC & _BV(ECHO)));
	TCCR1B |= _BV(CS12);

	while((PINC & _BV(ECHO)));
	TCCR1B &= ~_BV(CS12);
	
	return 340*(TCNT1 >> 1)/625;

}

void send_one() {
	
	asm (
	"ldi r18, 0b00100000	\n"
	"in r19, 0x0B			\n"
	"or r19, r18			\n"
	"out 0x0B, r19			\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"ldi r18, 0b11011111	\n"
	"in r19, 0x0B			\n"
	"and r19, r18			\n"
	"out 0x0B, r19			\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"ret					\n"
	);
	
}

void send_zero(){
	
	asm (
	"ldi r18, 0b00100000	\n"
	"in r19, 0x0B			\n"
	"or r19, r18			\n"
	"out 0x0B, r19			\n"
	"nop					\n"
	"ldi r18, 0b11011111	\n"
	"in r19, 0x0B			\n"
	"and r19, r18			\n"
	"out 0x0B, r19			\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"nop					\n"
	"ret					\n"
	);
	
}

ISR(USART0_RX_vect){
	recieveData = UDR0; // Recieve
	UDR0 = recieveData; // Transmit
}
```
