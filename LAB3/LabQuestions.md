# Chapter 1
_______________________________________________________________________________________________________________
• Use your delay code to create a program that blinks an LED at 5 Hz.

```
#define DDRB 0x04
#define PORTB 0x05
#define LED2 2
#define decreaser 139

start:
	ldi r18, (1<<LED2)
	ldi r24, (0<<LED2)
	in r17, DDRB
	or r17, r18
	out DDRB, r18
	in r17, PORTB
	or r17, r18


	call ledblink

end:
	rjmp end


	delay:
		ldi r20, decreaser
			l1:
			ldi r21, decreaser
				l2:
				ldi r22, decreaser
					l3:
					dec	r22
					brne l3
					dec r21
					brne l2
					dec r20
					brne l1
	ret

	ledon:
		ldi r18, (1<<LED2)
		in r17, DDRB
		or r17, r18
		out DDRB, r18
		in r17, PORTB
		or r17, r18
		out PORTB, r18
	ret

	ledblink:

		out PORTB, r18
		call delay
		out PORTB, r24
		call delay
	rjmp ledblink
```
_______________________________________________________________________________________________________________
## 1.1
How can you use your delay code in order to create arbitrary delays? What is the limitation?
  
  The limitation is the amount of bits used. Now we can only use 24 bits(three registers) so the limitation is 255*3
_______________________________________________________________________________________________________________
• Make the LED blink at 1 Hz instead.
  
  set decreaser to 81 instead
_______________________________________________________________________________________________________________
# Chapter 2
_______________________________________________________________________________________________________________
• Write an assembly subroutine that allocates an array of 5 bytes on the stack.

```
#define STACK_H 0x3E ; Address to the high byte of the stack pointer
#define STACK_L 0x3D ; Address to the low byte of the stack pointer
#define N_ALLOC 5

PROLOGUE:
	in r28, STACK_L ; Load low byte of stack pointer to r28
	in r29, STACK_H ; Load high byte of stack pointer to r29
	sbiw Y, N_ALLOC ; Subtract N_ALLOC from the loaded stack pointer
	out STACK_L , r28 ;
	out STACK_H , r29 ; Update stack pointer
ret
```
_______________________________________________________________________________________________________________
## 2.1
What is the value of the stack pointer directly after allocation? Use the debugger.
  
  0x40FA
_______________________________________________________________________________________________________________
• In the subroutine, sample the state of the buttons B1 to B6 in a loop and save the five latest in the
array. The first sample shall be placed at index zero (arr[0]), the fifth sample at index four. After
saving the values, simply return from the subroutine.
• In the main function, call your subroutine in a forever-loop.
• Start the debugger and step through each sample to verify that you can read the values of the buttons.
Also check the stack to verify that the samples are placed at the correct addresses.

```
#define STACK_H 0x3E
#define STACK_L 0x3D
#define N_ALLOC 5
#define PINA 0x00

start:
	call SUBROUTINE
	rjmp start

SUBROUTINE:
	in r28, STACK_L ; Load low byte of stack pointer to r28
	in r29, STACK_H ; Load high byte of stack pointer to r29
	sbiw Y, N_ALLOC ; Subtract N_ALLOC from the loaded stack pointer
	out STACK_L, r28 ;
	out STACK_H, r29 ; Update stack pointer

	ldi r20, 5
	loop:
		in r18, PINA
		and r19, r18
		std Y+1, r19
		dec r20
	brne loop

	in r28, STACK_L ; Load low byte of stack pointer to r28
	in r29, STACK_H ; Load high byte of stack pointer to r29
	adiw Y, N_ALLOC ; Add N_ALLOC to the loaded stack pointer
	out STACK_L, r28
	out STACK_H, r29
ret
```
_______________________________________________________________________________________________________________
## 2.2
What happens if we do not return the allocated memory in the subroutine?
_______________________________________________________________________________________________________________
## 2.3
Do we need to store the values in the allocated array? What happens if we just store the values
randomly on the stack?
_______________________________________________________________________________________________________________
# Chapter 3
_______________________________________________________________________________________________________________
• Write a program that blinks an LED if a button is pressed. That is, if a button is pressed, you shall call
led_on() and led_off() with a delay after both functions, use 500 ms. Note that the two functions
are not yet implemented, thus you will need to implement them in assembly. On the top of your C-file,
add

– extern void led_on(char),
– extern void led_off(char),
– extern char check_button(char).

• Declare the three subroutines and implement them following all conventions.
```
#define DDRB 0x04
#define PORTB 0x05
#define PINA 0x00

.global led_on
.global led_off
.global check_button

led_on:
	ldi r25, 0xff
	out DDRB, r25
	in r22, PORTB
	or r22, r24
	out PORTB, r22
ret

led_off:
	ldi r22, 255
	eor r24, r22
	ldi r25, 0xff
	out DDRB, r22
	in r22, PORTB
	and r22, r24
	out PORTB, r22
ret

check_button:
	in r18, PINA
	and r24, r18
ret


#define F_CPU 16000000UL
#include <util/delay.h>

extern void led_on(char);
extern void led_off(char);
extern char check_button(char);

int main(void)
{
    while (1) 
    {
		led_on(check_button(0b11111111));
		_delay_ms(500);
		led_off(check_button(0b11111111));
    }
}
```
_______________________________________________________________________________________________________________
## 3.1
When calling the subroutine, led_on, what is being pushed to the stack and why?
  
  We push witch button is beeing pressed aka r24 with a value of 0bxxxxxxxx.
_______________________________________________________________________________________________________________
## 3.2
When the ret instruction is executed, what happens with the stack pointer?
  
  The pointer points to the last item in the stack.
_______________________________________________________________________________________________________________
## 3.3
In the check_button subroutine, comment out the instruction where you place the return value
in r24 and run the code. Does the LED blink? If so, why?
  
  No, but something is wrong with our code, help needed
_______________________________________________________________________________________________________________
## 3.3.2 Hello, Mr. Anderson
```
#define PORTD 0x0B
 #define PIND 5
 #define PINA 0x00

 .global read_buttons
 .global send_one
 .global send_zero
 .global send_reset 
 .extern button_state

send_one: 

	ldi r20, (1<<PIND)
	in  r21, PORTD
	or  r21, r20
	out PORTD,r21
// Delay 12 cc
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop

	ldi r20, (1<<PIND)
	eor r21, r20
	out PORTD, r21

// Delay 12 cc
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
ret
	
send_zero:
	 ldi r20, (1<<PIND)
	 in  r21, PORTD
	 or  r21, r20
	 out PORTD, r21
// Delay
	 nop

	 ldi r20, (1<<PIND)
	 eor r21, r20
	 out PORTD, r21

// Delay
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop
	nop

ret

 send_reset:

    ldi  r18, 2
    ldi  r19, 9
	L1: dec  r19
		brne L1
		dec  r18
		brne L1
ret

 read_buttons:
	in r20, PINA
	lsr r20
	lsr r20
	sts button_state, r20
ret
```
_______________________________________________________________________________________________________________
# Chapter 4
_______________________________________________________________________________________________________________
## 4.1
sends a string to the OLED
_______________________________________________________________________________________________________________
## 4.2 
No it does not
_______________________________________________________________________________________________________________
## 4.3
It displays the string "H4ck3d" to the oled
_______________________________________________________________________________________________________________
## 4.4
pointer moves from 0x40DF to 0x40D1 = 14 steps
_______________________________________________________________________________________________________________
## 4.5
r22 and r24
_______________________________________________________________________________________________________________
## 4.6
We end up at hacked, which means this code is executed at the stack instead of secure function.
_______________________________________________________________________________________________________________
