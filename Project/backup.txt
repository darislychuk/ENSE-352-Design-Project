; Author of Whack-A-Mole - Daris Lychuk
; SID: 200361245
; ENSE 352
; December 6th, 2018
;;; Directives
            PRESERVE8
            THUMB       

        		 
;;; Equates

INITIAL_MSP	EQU		0x20001000	; Initial Main Stack Pointer Value


;PORT A GPIO - Base Addr: 0x40010800
GPIOA_CRL	EQU		0x40010800	; (0x00) Port Configuration Register for Px7 -> Px0
GPIOA_CRH	EQU		0x40010804	; (0x04) Port Configuration Register for Px15 -> Px8
GPIOA_IDR	EQU		0x40010808	; (0x08) Port Input Data Register
GPIOA_ODR	EQU		0x4001080C	; (0x0C) Port Output Data Register
GPIOA_BSRR	EQU		0x40010810	; (0x10) Port Bit Set/Reset Register
GPIOA_BRR	EQU		0x40010814	; (0x14) Port Bit Reset Register
GPIOA_LCKR	EQU		0x40010818	; (0x18) Port Configuration Lock Register

;PORT B GPIO - Base Addr: 0x40010C00
GPIOB_CRL	EQU		0x40010C00	; (0x00) Port Configuration Register for Px7 -> Px0
GPIOB_CRH	EQU		0x40010C04	; (0x04) Port Configuration Register for Px15 -> Px8
GPIOB_IDR	EQU		0x40010C08	; (0x08) Port Input Data Register
GPIOB_ODR	EQU		0x40010C0C	; (0x0C) Port Output Data Register
GPIOB_BSRR	EQU		0x40010C10	; (0x10) Port Bit Set/Reset Register
GPIOB_BRR	EQU		0x40010C14	; (0x14) Port Bit Reset Register
GPIOB_LCKR	EQU		0x40010C18	; (0x18) Port Configuration Lock Register

;The onboard LEDS are on port C bits 8 and 9
;PORT C GPIO - Base Addr: 0x40011000
GPIOC_CRL	EQU		0x40011000	; (0x00) Port Configuration Register for Px7 -> Px0
GPIOC_CRH	EQU		0x40011004	; (0x04) Port Configuration Register for Px15 -> Px8
GPIOC_IDR	EQU		0x40011008	; (0x08) Port Input Data Register
GPIOC_ODR	EQU		0x4001100C	; (0x0C) Port Output Data Register
GPIOC_BSRR	EQU		0x40011010	; (0x10) Port Bit Set/Reset Register
GPIOC_BRR	EQU		0x40011014	; (0x14) Port Bit Reset Register
GPIOC_LCKR	EQU		0x40011018	; (0x18) Port Configuration Lock Register

;Registers for configuring and enabling the clocks
;RCC Registers - Base Addr: 0x40021000
RCC_CR		EQU		0x40021000	; Clock Control Register
RCC_CFGR	EQU		0x40021004	; Clock Configuration Register
RCC_CIR		EQU		0x40021008	; Clock Interrupt Register
RCC_APB2RSTR	EQU	0x4002100C	; APB2 Peripheral Reset Register
RCC_APB1RSTR	EQU	0x40021010	; APB1 Peripheral Reset Register
RCC_AHBENR	EQU		0x40021014	; AHB Peripheral Clock Enable Register

RCC_APB2ENR	EQU		0x40021018	; APB2 Peripheral Clock Enable Register  -- Used

RCC_APB1ENR	EQU		0x4002101C	; APB1 Peripheral Clock Enable Register
RCC_BDCR	EQU		0x40021020	; Backup Domain Control Register
RCC_CSR		EQU		0x40021024	; Control/Status Register
RCC_CFGR2	EQU		0x4002102C	; Clock Configuration Register 2
	
RTC_CNTH	EQU		0x40002818	;Counter register


; Times for delay routines
        
DELAYTIME	EQU		9000;1021		; (200 ms/24MHz PLL)
ENDTIME	EQU		4000000		;0.5 second delay

; Values for random number generator

a			EQU		1664525
c			EQU		1013904223

; Vector Table Mapped to Address 0 at Reset
            AREA    RESET, Data, READONLY
            EXPORT  __Vectors

__Vectors	DCD		INITIAL_MSP			; stack pointer value when stack is empty
        	DCD		Reset_Handler		; reset vector
			
            AREA    MYCODE, CODE, READONLY
			EXPORT	Reset_Handler
			ENTRY

Reset_Handler		PROC

		BL GPIO_ClockInit
		BL GPIO_init
		
		LDR R4, = 0x0
		LDR R1, = 0x0
		LDR R11, = 0x0
	
waitLoop
		LDR R10, = DELAYTIME
		
		BL waitingLeds
		
		BL waitingButtons
		
		
		LDR R6, = GPIOA_ODR
		LDR R7,[R6]
		EOR R7,#0xFFFFFFFF
		AND R7,#0xFFFFE1FF			
		ORR R7,#0x0000
		EOR R7,#0xFFFFFFFF
		STR R7,[R6]
		CMP R3,#0xF
		BEQ waitLoop		
		
		BL mainGame
		
userWaitLoop
		CMP R3,#0xF
		BEQ userWaitLoop
		
	
mainLoop
		BL winningLeds
	
		B	mainLoop
		ENDP
			


;;;;;;;;Subroutines ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
	ALIGN
delay3 PROC
	PUSH{R10}
delayLoop3
	
	
	CMP R10,#0					;compare R10 to 0 (delaytime function)
	BLE doneDelay3				;branch to doneDelay routine so that it exits the delay loop
	SUB R10,#1					;subtract r10 by 2
	
	B delayLoop3				;branch back to the delayloop so the leds continue to flash
	
doneDelay3	
	POP{R10}
	;B buttonLoop
	BX LR 
	ENDP

	ALIGN
delay2 PROC
	LDR R10, = 800000
delayLoop2
	
	
	CMP R10,#0					;compare R10 to 0 (delaytime function)
	BLE doneDelay2				;branch to doneDelay routine so that it exits the delay loop
	SUB R10,#1					;subtract r10 by 2
	
	B delayLoop2				;branch back to the delayloop so the leds continue to flash
	
doneDelay2	
	;POP{R10}
	;B buttonLoop
	BX LR 
	ENDP
	
	ALIGN
delay PROC
	PUSH{R10}
delayLoop
	
	
	CMP R10,#0					;compare R10 to 0 (delaytime function)
	BLE doneDelay				;branch to doneDelay routine so that it exits the delay loop
	SUB R10,#1					;subtract r10 by 2
	
	B delayLoop					;branch back to the delayloop so the leds continue to flash
	
doneDelay	
	POP{R10}
	B buttonLoop
	BX LR 
	ENDP
	
;This routine will enable the clock for the Ports that you need	
	ALIGN
GPIO_ClockInit PROC

	; Students to write.  Registers   .. RCC_APB2ENR
	; ENEL 384 Pushbuttons: SW2(Red): PB8, SW3(Black): PB9, SW4(Blue): PC12 *****NEW for 2015**** SW5(Green): PA5
	; ENEL 384 board LEDs: D1 - PA9, D2 - PA10, D3 - PA11, D4 - PA12
	LDR R6,= RCC_APB2ENR		;load the clock address with offset into R0, address of the register
	LDR R0,[R6]					;load the register address into a register that can compare the bits
	ORR R0,#0x1C				;or R1 with bits 0001 and 0100 because you want bits 2 for port A and 4 for port C
	STR R0,[R6]					;store the bits back into the register address
	BX LR
	ENDP
		
	
	
;This routine enables the GPIO for the LED's.  By default the I/O lines are input so we only need to configure for ouptut.
	ALIGN
GPIO_init  PROC
	
	; ENEL 384 board LEDs: D1 - PA9, D2 - PA10, D3 - PA11, D4 - PA12
	LDR R6, = GPIOA_CRH
	LDR R0,[R6]
	LDR R1,= 0xFFF0000F
	AND R0,R1				;Specify the LED bits that will be used
	LDR R1,= 0x00033330
	ORR R0,R1
	STR R0,[R6]
	
	
    BX LR
	ENDP
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; This is the main subroutine for my game. This is where the random LED is turned on and where the user must input the
;;; proper button in order to continue
;;; Require:
;;; R5: Counter to get a random generated number for the LED game sequence
;;; R6: Load in the Input address of the pushbuttons in this register
;;; R7: Load in the value to clear and set certain LEDs
;;; R10: Used as delay loop register
;;; R12: Used as a register to count what round the user is at
;;; R9: Used as a register to store constant c
;;; R8: Used as a register to store constant a
;;; R2: Used as a register to decrement how much time the user has to push the button before they are sent to the end failure phase1
;;; Promise:
;;; returns a value in R6 which turns on one of the random LEDs and accepts user button input if it is the correct corresponding
;;; button to that LED. Also has a timer for how long the user actually has to push a button for the light
;;;
;;; Modifies:
;;; R11, R7, R12, R10, R0, R1, R2, R3, R8, R9		
	ALIGN
mainGame  PROC
	LDR R12,=0x0
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF		;Clear all bits and set the LEDs off
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0000
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	
	LDR R10, = 400000
	BL delay
	
randomLoop
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF		;Clear all bits when the user presses the button for the LED and set the LEDs off
	ORR R7,#0x0000
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	
	CMP R12,#0xF			;Register to count what level the player is at
	BEQ mainLoop			;If they hit 15 rounds, branch to winningLEDs
	ADD R12,#0x1			;Increment the level everytime the player runs through
	SUB R10,#0xFFF			;Subtract an amount from the delay
	LDR R8, = a				;Constant declared above used for random LED equation
	LDR R9, = c				;Constant declared above used for random LED equation
	MUL R8, R8, R5			;Part of random LED equation
	ADD R8, R8, R9			;Part of random LED equation
	LSR R8, #30				;Take the two MSB from that random number to decide which LED to turn on
	
	BL delay2
	
	CMP R8, #0x00			;Compare register R8 which contains the 2 bits that are being looked at for the random LED
	BEQ random1
	CMP R8, #0x01
	BEQ random2
	CMP R8, #0x02
	BEQ random3
	CMP R8, #0x03
	BEQ random4
random1
	LDR R6, = GPIOA_ODR		;If you are here it will turn the first light on
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF		
	ORR R7,#0x0200			;Clear and set the bits wanted to turn on first LED
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R5, R5				;Add R5(the random value based on when the user presses the first button) to itself
	LDR R2,=0x111111		;Load a value into R2 that will be decremented based on the level the player is on, this is used for how long the LED stays on
	B user1B				;Branch to corresponding button check based on LED

	BX LR
	
random2
	LDR R6, = GPIOA_ODR		;If you are here it will turn the second light on
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0400			;Clear and set the bits wanted to turn on second LED
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R5, R5				;Add R5(the random value based on when the user presses the first button) to itself
	LDR R2,=0x111111		;If wanting an easier time playing the game but still somewhat challenging, change the value to 0xFFFFF
	B user2B				;Branch to corresponding button check based on LED

	BX LR

random3
	LDR R6, = GPIOA_ODR		;If you are here it will turn the third light on
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF	
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0800			;Clear and set the bits wanted to turn on third LED
	EOR R7,#0xFFFFFFFF	
	STR R7,[R6]
	ADD R5, R5				;Add R5(the random value based on when the user presses the first button) to itself
	LDR R2,=0x111111	
	B user3B				;Branch to corresponding button check based on LED

	BX LR
	
random4
	LDR R6, = GPIOA_ODR		;If you are here it will turn the fourth light on
	LDR R7,[R6]					
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x1000			;Clear and set the bits wanted to turn on fourth LED
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R5, R5 				;Add R5(the random value based on when the user presses the first button) to itself
	LDR R2,=0x111111
	B user4B				;Branch to corresponding button check based on LED
	
	BX LR
	B randomLoop

user1B	
		SUB R2,R12			;Subtract register with initial value by the level value the player is on
		CMP R2,#0x0			;If this value reaches to 0 before the user can press the button, branches to losing phase
		BEQ losingLeds
		LDR R3,=0x0			;Load in a 0 so that it checks it everytime
		LDR R6,= GPIOB_IDR	;Port B 8 and 9
		LDR R0,[R6]
		LDR R1,= 0x00000100
		AND R0,R1
		LSR R0,#8
		ORR R3,R0
		CMP R3,#0x1
		BEQ user1B			;The wrong button was pressed or no buttons so repeat loop
		CMP R3,#0x0			;If the proper button was pressed branch back to randomLoop for the next LED to turn on
		BEQ randomLoop
		
user2B	
		SUB R2,R12
		CMP R2,#0x0
		BEQ losingLeds
		LDR R3,=0x0
		LDR R6,= GPIOB_IDR	;Port B 8 and 9
		LDR R0,[R6]
		LDR R1,= 0x00000200
		AND R0,R1
		LSR R0,#9
		ORR R3,R0
		CMP R3,#0x1
		BEQ user2B
		CMP R3,#0x0
		BEQ randomLoop
		
user3B	
		SUB R2,R12
		CMP R2,#0x0
		BEQ losingLeds
		LDR R3,=0x0
		LDR R6,= GPIOC_IDR	;Port C 12
		LDR R0,[R6]
		LDR R1,= 0x00001000
		AND R0,R1
		LSR R0,#12
		ORR R3,R0
		CMP R3,#0x1
		BEQ user3B
		CMP R3,#0x0
		BEQ randomLoop

user4B	
		SUB R2,R12
		CMP R2,#0x0
		BEQ losingLeds
		LDR R3,=0x0
		LDR R6,= GPIOA_IDR	;Port A 5
		LDR R0,[R6]
		LDR R1,= 0x00000020
		AND R0,R1
		LSR R0,#5
		ORR R3,R0
		CMP R3,#0x1
		BEQ user4B
		CMP R3,#0x0
		BEQ randomLoop
doneRandom
		BX LR
		ENDP
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Changes LEDs during to show the binary result of profficiency level the user failed at then returns to waiting phase
;;; Require:
;;;	R4: Used as a counter to increment LED sequence
;;; R5: Counter to get a random generated number for the LED game sequence
;;; R6: Load in the Input address of the pushbuttons in this register
;;; R7: Load in the value to clear and set certain LEDs
;;; R11: Load in 0 then use as a counter register for how many times to show winning LED sequence
;;; R10: Used as delay loop register
;;; Promise:
;;; returns a value in R6: this value is whether the LEDs are on or off
;;;
;;; Modifies:
;;; R11, R7, R12		
	ALIGN
losingLeds  PROC
	LDR R11,=0x0			;Restart counter in R11
	LSL R12,#9				;Shift user profficiency level over so that the bits align with the corrsponding LED bits

caseL1
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF		;Display user profficiency level here
	AND R7,#0xFFFFE1FF
	ORR R7,R12
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R11,#0x1
	BL delay2
	
caseL2
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF		;Have all lights shut off in second case so that LEDs flash in binary sequence for the players profficiency level
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0000
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	BL delay2				;Delay so that lights stay off long enough for the eye to see
	CMP R11,#0x5
	BNE caseL1				;Repeat the loop is the count has not been reached yet
	
	B waitLoop				;Go back to waiting phase if count has been reached
	
    BX LR
	ENDP		
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Changes LEDs to show winning sequence then waits for user input to restart or approximately 1 minute
;;; Require:
;;;	R4: Used as a counter to increment LED sequence
;;; R5: Counter to get a random generated number for the LED game sequence
;;; R6: Load in the Input address of the pushbuttons in this register
;;; R7: Load in the value to clear and set certain LEDs
;;; R11: Load in 0 then use as a counter register for how many times to show winning LED sequence
;;; R10: Used as delay loop register
;;; Promise:
;;; returns a value in R6: this value is whether the LEDs are on or off
;;;
;;; Modifies:
;;; R11, R10, R3, R0, R1		
	ALIGN
winningLeds PROC
	LDR R11, = 0x0			;Loads in 0 into R11 to set an empty register
	
case1W
	LDR R6, = GPIOA_ODR		;Port A 9
	LDR R7,[R6]				;Load that address into another register
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF		
	ORR R7,#0x1200
	EOR R7,#0xFFFFFFFF		;R7 used to clear and set bits needed to turn on the light
	STR R7,[R6]
	BL delay2				;Branch to delay so that the specific lights remain visible for a set time period

case2W
	LDR R6, = GPIOA_ODR		;Port A 10
	LDR R7,[R6]				;Load that address into another register
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0C00
	EOR R7,#0xFFFFFFFF		;R7 used to clear and set bits needed to turn on the light
	STR R7,[R6]
	ADD R11,#0x1			;Increment R11 count for how many cycles the winning lights rotate for
	BL delay2				;Branch to delay so that the specific lights remain visible for a set time period
	CMP R11,#0x5			;Compare the count register to 7 because I wanted 5 cycles before showing the players profficiency level
	BNE case1W				;If the count is not 5 branch back to the previous case so that the lights rotate

	
case3W
	LDR R6, = GPIOA_ODR		;Port A 11
	LDR R7,[R6]				;Load that address into another register
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			
	ORR R7,#0xFFFFE1FF		;R7 used to clear and set bits needed to turn on the light
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
		
	BL delay2				;Branch to delay again to have lights remain on for set time
	LDR R10, = 0x3700		;Approximately 1 minute delay if no buttons are pressed before returning to waiting(UC2) phase

	LDR R6, = GPIOA_ODR		;Port A 12
	LDR R7,[R6]				;Load that address into another register
	EOR R7,#0xFFFFFFFF		
	AND R7,#0xFFFFE1FF			
	ORR R7,#0xFFFFFFFF		;orr with whatever level they finished at
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
case4W

	SUB R10,#0x1			;Subtract delay count in order to show profficiency level for approximately 1 minute
	
	BL delay3				;Branch to delay subroutine
	
	LDR R3,=0x0
	
	LDR R6,= GPIOB_IDR	;Port B 8 and 9
	LDR R0,[R6]
	LDR R1,= 0x00000300
	AND R0,R1
	LSR R0,#8
	ORR R3,R0
		
	LDR R6,= GPIOC_IDR	;Port C 12
	LDR R0,[R6]
	LDR R1,= 0x00001000				;;These three groupings determines if any buttons are pressed while showing the profficiency level
	AND R0,R1
	LSR R0,#10
	ORR R3,R0
		
	LDR R6,= GPIOA_IDR	;Port A 5
	LDR R0,[R6]
	LDR R1,= 0x00000020
	AND R0,R1
	LSR R0,#2
	ORR R3,R0

	CMP R3,#0xF

	BNE waitLoop		;If any button is pressed, start another new game

	CMP R10,#0x0
	BNE case4W			;If the time runs out go back to waiting phase

	B waitLoop
	
	BX LR
	LTORG
	ENDP
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Changes LEDs during waiting phase
;;; Require:
;;;	R4: Used as a counter to increment LED sequence
;;; R5: Counter to get a random generated number for the LED game sequence
;;; R6: Load in the Input address of the pushbuttons in this register
;;; R7: Load in the value to clear and set certain LEDs
;;; Promise:
;;; returns a value in R6: which turns on a specific LED
;;;
;;; Modifies:
;;; R4, R5	
	ALIGN
waitingLeds PROC
		
	ADD R5, #0x555		;R5 used as register to increment for random seed later used in the game
	CMP R4, #0x0		;Compare register R4 count to determine which LED should be on
	BEQ case1
	CMP R4, #0x1
	BEQ case2
	CMP R4, #0x2
	BEQ case3
	CMP R4, #0x3
	BEQ case4
case1
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF		;R7 used to clear and set any bits wanted
	ORR R7,#0x0200
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R4, #0x1
	
	BX LR
	
case2
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0400			;R7 used to clear and set any bits wanted
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	ADD R4, #0x1
	
	BX LR

case3
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]
	EOR R7,#0xFFFFFFFF	
	AND R7,#0xFFFFE1FF			
	ORR R7,#0x0800			;R7 used to clear and set any bits wanted
	EOR R7,#0xFFFFFFFF	
	STR R7,[R6]
	ADD R4, #0x1

	BX LR
	
case4
	LDR R6, = GPIOA_ODR
	LDR R7,[R6]					
	EOR R7,#0xFFFFFFFF
	AND R7,#0xFFFFE1FF			;AND R7 by the compliment to clear all the bits
	ORR R7,#0x1000				;ORR R7 by the compliment of 0xC to turn the bits on(active low)
	EOR R7,#0xFFFFFFFF
	STR R7,[R6]
	LDR R4,= 0x0
	
	BX LR
	ENDP
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Wait for user input(press of 1 of the 4 buttons)
;;; Require:
;;;	R0: Used to clear then shift the bits being looked at for button input
;;; R3: Load in 0 initially then store if a button is pressed in this register
;;; R6: Load in the Input address of the pushbuttons in this register
;;; R1: Load in the value checking to see if a button is pushed into this register.
;;; This will be ANDed with R0 which is then ORRed with R3 to set the bits being looked at 
;;;	and then store if a button has been pushed
;;; R10: Used as a delay loop register
;;; Promise:
;;; returns a value in R3: (F, if nothing is pushed, a different value if a button is pushed)
;;;
;;; Modifies:
;;; R0, R1, R3, and R10	
	ALIGN
waitingButtons PROC
		PUSH{R10}
buttonLoop
		
		
		LDR R3,=0x0			;Clear the R3 register everytime
		
		LDR R6,= GPIOB_IDR	;Port B 8 and 9
		LDR R0,[R6]			;Load that address into another register
		LDR R1,= 0x00000300	;Load the hex value of the 300 to check and see if either button 1 or 2 from the left are pushed
		AND R0,R1			;Clear the bits I dont want
		LSR R0,#8			;Shift the bits over to the most right position(LSB)
		ORR R3,R0			;Set the bits I want into R3
		
		LDR R6,= GPIOC_IDR	;Port C 12
		LDR R0,[R6]			;Load that address into another register
		LDR R1,= 0x00001000	;Load the hex value of the 1000 to check and see if either button 3 from the left are pushed
		AND R0,R1			;Clear the bits I dont want
		LSR R0,#10			;Shift the bits over to the most right position(LSB) the is not already being used
		ORR R3,R0			;Set the bits I want into R3
		
		LDR R6,= GPIOA_IDR	;Port A 5
		LDR R0,[R6]			;Load that address into another register
		LDR R1,= 0x00000020	;Load the hex value of the 20 to check and see if either button 3 from the left are pushed
		AND R0,R1			;Clear the bits I dont want
		LSR R0,#2			;Shift the bits over to the most right position(LSB) the is not already being used
		ORR R3,R0			;Set the bits I want into R3
	
		CMP R3,#0xF			;Compare all values in R3 to F to check if any of the buttons are being pressed
		
		BNE doneButtonLoop	;If not equal that means the one of the buttons is pushed therefore branch out
		
		SUB R10,#1			;Decrement my delay count
		CMP R10,#0			;Compare my delay count to 0 to know when to light up the next light
		
		BNE buttonLoop		;If R10 is not equal to 0 repeat the loop so that the light is on for a set amount of time

doneButtonLoop
		POP{R10}			;Get out of the loop if a button is pressed or when the delay is 0 to go to a different light
						
		BX LR
		ENDP

	ALIGN
	END