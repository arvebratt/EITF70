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
	
	It is now 1.
	
	
	
```
#include <avr/io.h>
#include <avr/interrupt.h>
#define interuptPins PCMSK2
#define S1Interupt PCINT23
#define S2Interupt PCINT22

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

ISR(PCINT2_vect){
	//my cage code
	
	//before dawn when the lions sleeps tonight.....
	if(/*read1 &&*/ !goesOut || !goesIn){
		goesOut = 1;
	}
	if(/*read2 &&*/ !goesOut || !goesIn){
		goesIn = 1;
	}
	
	if(goesOut){
		if(/*read båda ||*/ middle){
			middle = 1;
			if(/*!read2*/){
				lions++;
				middle = 0;
				goesOut = 0;
			}
		}
	}
	if(goesIn){
		if(/*read båda ||*/ middle){
			middle = 1;
			if(/*!read1*/){
				lions++;
				middle = 0;
				goesIn = 0;
			}
		}
	}
	
	send_lions(lions);
}
```
