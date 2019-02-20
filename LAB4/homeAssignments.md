```
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>

#define S1 PCINT22
#define S2 PCINT23



int main(void)
{
	sensor_init();
    while (1) 
    {
		_delay_ms(500)
		if(S2 == 1 && S1 == 0){
			lion_goes_out();
		} else if (S1 == 1 && S2 == 0){
			lion_goes_in();
		} 
		
    }
}

void sensor_init(){
	DDRC = (0<<DDRC6) | (0<<DDRC7)
}

void lion_goes_out(){
	if(S1 == 1 && S2 == 1){
		if(S1 == 1 && S2 == 0){
			if(S1 == 0 && S2 == 0){
				//lion now out
			}
		}
	}
		//break
}

void lion_goes_in(){
	if(S1 == 1 && S2 == 1){
		if(S1 == 0 && S2 == 1){
			if(S1 == 0 && S2 == 0){
			//lion now in
			}
		}
	}
	//break
}
```
