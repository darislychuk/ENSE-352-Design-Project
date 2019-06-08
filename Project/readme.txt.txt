Daris Lychuk
200361245
ENSE 352
December 6th,2018

1. What the game is?

1) This is a whack-A-mole like game where leds are used instead of the moles, and pushbuttons are
used instead of a hammer. The goal of the game is to push the pushbutton corresponding to the led
that is currently illuminated.

2. How to play

2) To play the whack-A-mole that I have created, the user must hit the reset button in order to go
into phase 2 which is the Waiting phase that has a sequence of leds going off in a certain pattern.
The pattern that I chose to do was having my leds flash left to right very fast. The user must press
any of the 4 coloured buttons(red, black, blue, green) on the main board in order to proceed to the
next phase, which is the game phase.

In the game phase, all of the leds are shut off with a slight delay before anything happens. Once the
delay(approximately a second or two) is done, the first random LED comes on. The user must then push
the assigned button for the LED to turn off and proceed further along in the game. I have implemented
up to 15 levels currently.

If the player is able to press all of the corresponding pushbuttons in time and properly,
the player has won the game sending themselves to the End Success phase. In this phase,the two 
outter most and inner most LEDS will rotate in a pattern for a few cycles before displaying the
players profficiency level, which in this case will always be 15 if in the End Success phase,
therefore displaying all 4 LEDS on due to 15 being 0xF which in turn is 1111 in binary. While the
profficiency level is being displayed, the user has the ability to press any of the 4 pushbuttons to
start another game, if not the profficiency level stays lit for around a minute before automatically
returning to the Waiting phase.

If the player is unable to press all of the corresponding pushbuttons in time and/or properly, the
game is supposed to go into the End Failure phase which flashes the players profficiency level for
a few cycles before going back to the Waiting phase, allowing the user to start all over again, if
they wish.

3. Any information about problems encountered, features you failed to implement, extra features
you implemented beyond the basic requirements, possible future expansion, etc.

3)Problems encountered for myself would be trying my best to not have really messy code, unfortunately,
this is very easy for me to do unless I am given a greater time to clean it all up.
I had quite a bit of trouble implementing the branching to the End Failure phase properly.
The trouble I was having was getting it so that if the player pushed the wrong button or timed out, it
would send them there. Getting my End Failure phase to work was very easy and I was able to test it
by changing my branching to my End Success to End Failure instead, as well as changing what the max
number of levels were in order to test multiple values.
Features I failed to implement:
The features I failed to implement was if the user presses the wrong button. I started to implement
this into my code, however, ran out of time unfortunately. I got it so the first 2 LEDs followed
the rules of only allowing the corresponding pushbutton to pass them and the rest send them to a
losing phase instead of being able to try again. Here is just some of the logic I began to implement.
Unfortunately, it is not the cleanest code and is very easy to make a mistake. Even though it worked
for the first 2 LEDs for testing, I removed it due to just having everything looking the same.
user1B	
		
		SUB R2,R12		;Subtract register with initial value by the level value the player is on
		CMP R2,#0x0		;If this value reaches to 0 before the user can press the button, branches to losing phase
		BEQ losingLeds
		LDR R3,=0x0		;Load in a 0 so that it checks it everytime
		LDR R6,= GPIOB_IDR	;Port B 9
		LDR R2,[R6]
		LSR R2,#8
		AND R0,R2,#0x73		;;Testing if no buttons are pushed
		CMP R0,#0x73
		BEQ user2B
		AND R0,R2,#0x2		;;Testing if button 2 is pushed
		CMP R0,#0x2
		BEQ losingLeds		
		AND R0,R2,#0x10		;;Testing if button 4 is pushed
		CMP R0,#0x10
		BEQ losingLeds
		AND R0,R2,#0x20		;;Testing if button 3 is pushed
		CMP R0,#0x20
		BEQ losingLeds
		
		AND R0,R2,#0x1		;;Testing if button 1 (the correct button) is pushed
		CMP R0,#0x1
		BEQ randomLoop
user2B		
		SUB R2,R12		;Subtract register with initial value by the level value the player is on
		CMP R2,#0x0		;If this value reaches to 0 before the user can press the button, branches to losing phase
		BEQ losingLeds
		LDR R3,=0x0		;Load in a 0 so that it checks it everytime	
		LDR R6,= GPIOB_IDR	;Port B 9
		LDR R2,[R6]
		LSR R2,#8
		AND R0,R2,#0x73		;;Testing if no buttons are pushed
		CMP R0,#0x73
		BEQ user2B
		AND R0,R2,#0x1		;;Testing if button 1 is pushed
		CMP R0,#0x1
		BEQ losingLeds		
		AND R0,R2,#0x10		;;Testing if button 4 is pushed
		CMP R0,#0x10
		BEQ losingLeds
		AND R0,R2,#0x20		;;Testing if button 3 is pushed
		CMP R0,#0x20
		BEQ losingLeds
		
		AND R0,R2,#0x2		;;Testing if button 2(the correct button) is pushed
		CMP R0,#0x2
		BEQ randomLoop
Even though not problems, some extra information would be:
If later on somebody would like to change how many max levels there are, the way I show the
profficiency level currently in the winning phase would have to be modified. I was unsure of
how I would have wanted to show more than 15 levels with the LEDs and thats why I left the
maximum at 15.
In the End Failure phase, depending on when the user failes, the LEDs actually print the binary
value backwords to what you expect when looking at the board upright. This would be another feature
if given a bit more time would be easy to fix.
Possible future expansion would be the idea of implementing multiple lights to go off at a given point
in time, or even having their current score being displayed on the LCD screen above, however, I would
think this would prove quite difficult for someone like myself, unless given more time.

4.How the user can adjust the game parameters, including:
(a) PrelimWait: at the beginning of a cycle, the time to wait before lighting a LED
(b) ReactTime, the time allowed for the user to press the correct button to avoid terminating
the game.
(c) The number of cycles in a game: NumCycles
(d) values of WinningSignalTime and LosingSignalTime.

4)
a)This is very easy to do, all the user has to do is change the value in the register R10, which is
the register I used to store the amount of time for which the delay to occur. One could also always
place another delay function or even take it out if they so wish.

b)You would have to change the value inserted into R2 in the mainGame. Currently it is set to 
0x111111, if wanting an easier game, but still challenging, change value to 0xFFFFF

c)This is also very easy to do. R12 is the register I use to increment which level the player is on.
They would simply have to change the compare in my mainGame to a greater number than 0xF(15 decimal)
if they wish to play more levels in one go.

d)End Success and End Failure both work very similar. If wanting to change how long these singals are
displayed for, all someone would have to do is change R11, the register I used to count how many times
I wanted it to loop, to a greater value. They could also change the sequence of the winning light
pattern if they wanted by ignoring the first byte and looking at bits 1-4 after that byte.