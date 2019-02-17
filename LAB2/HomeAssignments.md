# Chapter 1
_______________________________________________________________________________________________________________
## 1.1
The LEDs are connected to Port B. The address to the output register is as shown in Table 1.2.
Using a pointer to the address, how do you turn LED number 3 (only) on?
```
int main(void)
{
	int *ledInit = 0x24;
	int *ledData = 0x25;
	while (1)
	{
		*ledInit |= (1<<3);
		*ledData |= (1<<3);
	}
}
```
____________________________________________________________________________________________________________
## 1.2
The buttons are connected to Port A. The address to the input register is as shown in Table 1.2.
Using a pointer to the address, how do you read the value of (only) button 4?
```
int main(void)
{
	int *buttonInit = 0x21;
	int *ledInit = 0x24;
	
	unsigned int current;
	unsigned int previous = 0;
	
    while (1) 
    {
		*buttonInit |= 0b00000000;
		*buttonData |= 0b001000;
    }
}
```
_______________________________________________________________________________________________________________
## 1.3
No, we do not, you could sample the switch signal at a regular interval and filter out the noice.
_______________________________________________________________________________________________________________
## 1.4
Setup a variable to act as a shift register, initialise it to xFF.
  Setup a regular sampling event, perhaps using a timer. Use a period of about 1ms.

Setup a variable to act as a shift register, initialise it to xFF.
Setup a regular sampling event, perhaps using a timer. Use a period of about 1ms.

On a sample event:
    Shift the variable towards the most significant bit
    Set the least significant bit to the current switch value
     if shift register val=0 then
      Set internal switch state to pressed
    else
      Set internal switch state to released
   end if
_______________________________________________________________________________________________________________
# Chapter 2
## 2.1 - 2.5
```
#define FOSC 1843200 // Clock Speed
#define BAUD 9600
#define MYUBRR FOSC/16/BAUD-1

#include <avr/io.h>
void usart0_init();
uint8_t usart0_receive();
void usart0_transmit(uint8_t data);

void main( void )
{
	usart0_init();
	while(1){
		usart0_transmit(0x41);
		uint8_t receiveData = usart0_receive();
		usart0_transmit(receiveData);
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
```
_______________________________________________________________________________________________________________
# Chapter 3
## 3.1 - 3.3
```
void timer3_init(){
	//Fast PWM with ICP3 as top
	TCCR3A = (1 << COM3A1)|(0 << COM3A0);
	TCCR3B = (1 << COM3B1)|(0 << COM3B0);

	//timer 3 with prescaler value 1024
	TCCR3B = (1 << CS32)|(0 << CS31)|(1 << CS30);

	//set PB6 as output
	PORTB = (1 << PCINT14);
	DDRB = (1 << DDB6);
}
```
_______________________________________________________________________________________________________________
# Chapter 5
## 5.1
0.5 * 2 / 340 ~= 2.9 milliseconds
_______________________________________________________________________________________________________________
## 5.2
0.003 * 16000000/65535 = 0.73 ~= 1
_______________________________________________________________________________________________________________
## 5.3
(13337 / 16000000) / 2 * 340 = 0.14 m
