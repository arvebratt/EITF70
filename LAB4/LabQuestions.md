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

