```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

#define S1 PORTC6
#define S2 PORTC7




int main(void)
{
	uint8_t state = 00;
	sensor_init();
	led_init();
	while (1)
	{	
		state = current_state(read_s1(), read_s2());
		led_toggle(state);
	}
}

void sensor_init(){
	DDRC |= (0<<DDRC6) | (0<<DDRC7);
}

int read_s1(){
	return PINC & (1<<S1);
}

int read_s2(){
	return PINC & (1<<S2);
}

//gör state för in eller ut
int current_state(int gate1, int gate2){
	int temp |= (gate2<<1) | (gate1<<0);
	return temp;
}





//test här
void led_init(){
	DDRB = 0xff;
}

void led_toggle(uint8_t check){
	if(check == 00){
		PORTB |= (1<<PCINT9);
	}
	if(check == 01){
		PORTB |= (1<<PCINT10);
	}
	if(check == 11){
		PORTB |= (1<<PCINT11);
	}
	if(check == 10){
		PORTB |= (1<<PCINT12);
	}
}
```
