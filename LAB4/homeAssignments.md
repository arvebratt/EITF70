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
