```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

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
		lionStart(read_s1(), goesOut, goesIn, 1);
		lionStart(read_s2(), goesIn, goesOut, 0);
		lionMiddle(read_s1(), read_s2(), goesOut, middle, lions, 1);
		lionMiddle(read_s2(), read_s1(), goesIn, middle, lions, 0);
		
	}
}

//function when lion starts its journey from the 
//den to the wild zone or the other way around
void lionStart(int read, int way1, int way2, int decider){
	if (read && (!way1 || !way2))
	{
		if(decider == 1){
			goesOut++;
		} else{
			goesIn++;
		}
		}
}

//function when lions are in the middle of their amazing journey
//plus it lights on the LEDs for our debugging pleasure
void lionMiddle(int read1, int read2, int way1, int way2, int lions, int decider){
	if (way1)
	{
		if ((read1 && read2) || way2)
		{
			way2 = 1;
			if (!read1)
			{
				led_toggle(lions);
				if(decider == 1){
					lions--;
				} else{
					lions++;
				}
				led_toggle(lions);
				middle = 0;
				if(decider == 1){
					goesIn = 0;
				} else{
					goesOut = 0;
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
