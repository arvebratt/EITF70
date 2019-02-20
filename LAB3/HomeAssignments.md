# Chapter 1
_______________________________________________________________________________________________________________
## 1.1
How many clock cycles are needed if a delay of 0.1 s is desired with a clock frequency of 16
MHz?

  1.6M clock cycles
_______________________________________________________________________________________________________________ 
## 1.2
How many bits do we need to count to the value above?

  21 bits
_______________________________________________________________________________________________________________
## 1.3
How many registers do we need for the delay of 0.1 s?

  3
_______________________________________________________________________________________________________________
## 1.4
Write a snippet of assembly code that delays the program for roughly 0.1 s. Remember that
the jump instructions also take time to execute.

```
	l1:
		dec r20
		brne l1
			dec r21
			brne l1
				dec r22
				brne l1
		nop
	ret  
```
_______________________________________________________________________________________________________________
## 1.5
How do you turn on LED 2 on port B in assembly? Remember to set the direction to an output.
Refer to Appendix A for instructions

```
#define DDRB 0x04
#define PORTB 0x05
#define LED2 2

start:
	  ldi r18, (1<<LED2)
	  in r17, DDRB
    or r17, r18
	  out DDRB, r18
    in r17, PORTB
	  or r17, r18
	  out PORTB, r18
end:
	  rjmp end
```
_______________________________________________________________________________________________________________
# Chaper 2
_______________________________________________________________________________________________________________
## 2.1
How do you allocate 10 integers on the stack
  
  change N_ALLOC to 10
_______________________________________________________________________________________________________________
## 2.2
If you allocate an array of bytes on the stack, and the stack pointer after allocation is 0x40E0,
what is the address of the value at index 3?
  
  0x40DE
_______________________________________________________________________________________________________________
# Chapter 3
_______________________________________________________________________________________________________________
## 3.1
In AVR assembly, how do you declare a subroutine? How do you call it?

  Declare it by:
```
subroutine:
	; code
	; something Rxx
ret ; returning Rxx
```
  and calling it with:
```
something something, Rxx
```
_______________________________________________________________________________________________________________
## 3.2
In AVR assembly, how are arguments passed to and returned from a subroutine?
  
  By saving and setting values in registers (Rxx)
_______________________________________________________________________________________________________________
## 3.3
Sending zero:
  0.35 microsecs = 5.6 clock cycles
  0.8 microsecs = 12.8 clock cycles
Sending one:
  0.70 microsec = 11.2 clock cycles
  0.60 microsecs = 9.6 clock cycles
Sending ret code:
  50 microsecs = 800 clock cycles
_______________________________________________________________________________________________________________
