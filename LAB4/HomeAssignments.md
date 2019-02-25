# Chapter 1
_______________________________________________________________________________________________________________________________________
## 1.1
How do you initialize the used I/O-pins as inputs? The pins can be found on port C, use the
schematic in Figure 2.
```
DDRC |= (0<<DDRC6) | (0<<DDRC7);
```
_______________________________________________________________________________________________________________________________________
## 1.2
Write code that polls the opto interrupters and counts the number of lions that are out in the
wild zone.

```
#define F_CPU 16000000UL
#include <avr/io.h>

#define S1 PORTC6
#define S2 PORTC7

uint8_t lions = 4;
int goesOut = 0;
int middle = 0;
int goesIn = 0;


int main(void)
{
	sensor_init();
	led_init();

	while (1){
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
					led_toggle(lions);
					lions++;
					led_toggle(lions);
					middle = 0;
					goesOut = 0;
				}	
			}
		}

		if(goesIn){
			if((read_s1() && read_s2()) || middle){
				middle = 1;
				if(!read_s1()){
					led_toggle(lions);
					lions--;
					led_toggle(lions);
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

int read_s2(){
	return PINC & (1<<S2);
}

//test här
void led_init(){
	DDRB |= 0xff;
}

void led_toggle(uint8_t check){
    PORTB ^= (1 << check);
}
```
__________________________________________________________________________________________________________________________________
# Chapter 2
__________________________________________________________________________________________________________________________________
## 2.1
Look on page 13 in the datasheet to see which PCINT (Pin Change Interrupt) the sensors are
connected.
	PC7 = PCINT23
	PC6 = PCINT22
__________________________________________________________________________________________________________________________________
## 2.2
Write code that enables the correct Pin Change Interrupt in the corresponding Pin Change
Mask Register, PCMSK. See page 92, 93, 94 and 95 in the datasheet for the details.
```
#define interuptPins PCMSK2
#define S1Interupt PCINT23
#define S2Interupt PCINT22

void interupt_init(){ 
	SREG |= (1<<7); // Enables interrupts on the microcontroller 
	PCICR |= (1<<PCIE2); // Since we have PCINT22/23, we use PCIE2 in PCICR to enable Pin change interrupt 
	interuptPins |= (1<<S1Interupt)|(1<<S2Interupt); // Enable the pins 
}
```
__________________________________________________________________________________________________________________________________
## 2.3
How do you enable the pin change interrupt for the corresponding port in the Pin Change
Interrupt Control Register, PCICR? See page 90 in the datasheet.
	
	The code above is an example of how we do that on PCIE2.

__________________________________________________________________________________________________________________________________
## 2.4
How do you enable the USART RX complete interrupt. For details please refer to the datasheet.
```
USCR0B |= (1<<RXCIE);
```
