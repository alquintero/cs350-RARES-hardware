# cs350-RARES-hardware
This is for our hardware team.

NOTE: To open .circ files:
1. click "Clone or Download"
2. click "Download Zip"
3. .circ file should be in a folder in the Downloads on your computer, just double click on it to open it with logisim

## DESIGN DOC

*For 3/3/19 Due Date.*

Our original idea for the hardware was to create a currently unused opcode for our putchar function and process it through the ALSU similarly to previously created functions in CS240. 

Two parameters: 
Eventually, we would take in input through the first keyboard but for now we are manually inputting input. The ASCII value of that character will go into R2. For string input, we are doing this character by character in accordance with the putchar function written by the software team (as opposed to taking in the string all at once). The ASCII value of each character, having been put into R2, would go into our ROM via HAssem and be taken in by our ALSU along with the opcode for putchar. The putchar circuit within the ALSU would simply return that same ASCII value and output it. From the ALSU, the ASCII value would go into R2. Then, R1 would hold our first terminal as input. 

As we worked with our microprocessor made last semester, we discovered there was much to do in order to make putchar possible. We also had to create Call and Return as our microprocessor did not currently have it. Then, we adjusted our original concept to make sure everything could flow in the correct paths and ultimately output into the terminal correctly. 

Currently, our microprocessor works for one character inputs and strings.

Notes for microprocessor:
- works on all lab 5 tests
- WORKS on lab 6 test 13 (has CBON, CCBOFF, MOVE) 
- WORKS on lab 6 test 14 (has CALL, RETURN)
- WORKS on lab 6 test 17 (has CALL, RETURN)
- WORKS on lab 6 test 18 (has CALL, RETURN)
- WORKS on lab 6 test 19 (has NESTED CALLS)

*For 3/26/19 Due Date.*

We implemented putchar and getchar using two currently unused opcodes. For intance, the opcode 0010 0100 tells the microprocessor we are performing putchar. The low 8 bits give the adresses of registers which determine where we look for information on which screen we are printing to and what character we are printing, respectively. For example, the code 0x2412 would call purchar, looking in R1 to see which terminal we are printing to (0 for terminal 1, 1 for terminal 2) and looking in R2 for ASCII character code. Now, the opcode 0010 0101 tells the microprocessor we are performing getchar. The low 8 bits give the addresses of registers which determine where we look for information on which keyboard we are getting characters from and where we are putting the character. For example, 0x2534 would call getchar, looking in R3 to see which keyboard we are taking input from (0 for terminal 1, 1 for terminal 2) and storing the input character into R4. Finally, we implemented two opcodes to check whether there is input in keyboard1 and keyboard2. The first is 0010 0110. The lowest 4 bits give the address of the register in which we will store the result. For instance, 0x2605 calls a check on keyboard1, storing 0 in R5 if there are NOT characters in keyboard1 or a 1 in R5 if there ARE characters in keyboard1. The second check code is 0010 0111. The lowest 4 bits give the address of the register in which we will store the result. For instance, 0x2706 call a check on keyboard2, storing 0 in R6 if there are NOT characters in keyboard2 or a 1 in R6 if there ARE characters in keyboard2. All of these data are written into the registers through multiplexers in the ALSU, which depend on 4 bit selectors that come from op control.

*For 4/28/19 Due Date.*

To implement interrupts, we let the hardware have most of the program control. Instead of checking when keyboards 1 and 2 had a character(s), we now had the keyboards automatically call the getchar+putchar functions if they had a character. 

Issues we came across + fixed:

- **Resetting registers that hold key locations in memory.** We still had our infinite loop with busy waiting that would set R6 to 0x1000 and R8 to 0x2000 for the start of keyboard_1_handler and keyboard_2_handler, respectively. 
- **Branching commands conflict.** We also ran into an issue with conflicting branching commands (within the interrupt handler of keyboard 2) and so we had to manually change some register uses (i.e. not only use R11 but R12 and R13 to keep track of where we need to branch to in certain cases).
- **Disallowing interrupts within interrupts.** In order to disallow this (as if we allowed it this could lead to starvation), we only allowed an interrupt to be called if:
1. the keyboard had a character (availability bit was on)
2. the machine code was in the infinite loop (we weren't in another interrupt)
3. the R6 register for the keyboard_1_handler = 1000 or the R8 register for the keyboard_2_handler = 2000 (for the corresponding interrupt requested)


* FOR FINAL LAB *

-----------------------------------------------------------------------------------

 New Opcodes:

280X — saving what PC+1 was during SWI (2200) to register X
290X — writing value in register X to PC

2200 — Software interrupt: switch threads!
2300 — Return from Software interrupt: back to main loop!

------------------------------------------------------------------------------------

Overview:

Locations 0x1000 and 0x2000 in ROM still contain the code for handling keyboard interrupts because I don’t want to lose functionality and all the work we did!

Location 0x3000 in ROM contains the software interrupt that switches context (aka saving current and loading in the new state). It accomplishes this by first saving current state at RAM location at 0x5000, then loading from 0x4000, then we move things from 0x5000 to 0x4000. Then we return. Also, I designed two new opcodes to give the HERA programmer a glance at the program counter and the ability to set it and save it (in this way, the programmer can write the interrupt code to fully save state).

------------------------------------------------------------------------------------
Simplistic way to think about the process:

This process is similar to the coding idiom:
temp = x;
x = y;
y = temp;

Temp is location 0x5000 in RAM, x is current state, and y is location 0x4000 in RAM.

------------------------------------------------------------------------------------

Conventions:

PC is stored in the first slot in RAM (so 5000 and 4000).
The 15 registers are stored in the next slots (X001 up to X00f). Note that we do not need to save and load the zero register.
The flags are saved in the last slot in RAM (X010).

------------------------------------------------------------------------------------

Method:

Interrupt code is called, so we go ahead and save the value of PC+1 in the next line counter so that later we can save this value as part of the state for this thread1.

Next, we jump to our interrupt code at 0x3000 in ROM. First we go ahead and store R1 at 0x0000 in RAM because we’ll be needing it to store at a deeper location in RAM. With that out of the way, we set R1 to 0x5000 so we can begin storing our state elements from thread1 at place 0x5000 in RAM. We store R2 through R15 in 0x5002 through 0x500f. Then we put the value of PC+1 from our initial interrupt (opcode for this is 0x280X, where X is the destination register) into R2 (so, 0x2802). We then store this at 0x5000 in RAM. We then get the flags into R2 and store them at 0x5010 in RAM. We’re done storing the current state! Now, we set R1 to 0x4000 so we can load from 0x4000 in RAM (this is where the state of thread2 is). We load the flags into R2 and then restore them. Then we load the PC+1 value into R2 and save that in a temporary register in the next line counter so that when we use RTI (Machine code 0x2300), we will return at PC+1 (where we left off). Finally, we load all the registers, starting at R2 (because we actually need R1 to be 0x4000 for the load statements) and going to R15, then doing R1. Now we have the state of thread 2 loaded into the processor! BUT, we need to copy the state we saved into 0x5000 through 0x5010 over to 0x4000 through 0x4010 so that it’s ready to be loaded when we need to make the next context switch. SO, we quickly store R1, R2, and R3 in 0x0000, 0x0001, and 0x0002 of RAM because we need those registers free to get our values from the 5000s to the 4000s (a series of loads and stores), and we don’t want to lose the actual values of R1 through R3 while using them temporarily. So, we set R1 to 0x4000 and R2 to 0x5000, and we proceed to load data from location 0x5000 into R3, then store R3 to 0x4000. Next we load data from location 0x5001 into R3, then store R3 to 0x4001. We continue in this way until it’s all copied over; then we’re done! So, we return from interrupt with machine code 0x2300, and our next line counter knows where to pick up with the thread we just loaded in.

------------------------------------------------------------------------------------

Thoughts on extensions:

Note: If we wanted to switch among more than two threads, we could achieve this by having the interrupt code 220X load up the thread state from RAM beginning at (a predetermined) location stored in register X. So we could just set R1 to be the location of thread 9, say 9000, and then use 2201 to load in state for thread 9 (at location 9000 in RAM). Then we would do the swap as described below!

------------------------------------------------------------------------------------

How to run the example:

Note: The keyboard no longer work because of memory and register conflicts, so don’t type stuff in there.

Starting at 0x4000 in RAM, enter d, 1, 2, 3, 4, 5, 6, 7, 8, 9, a, 4, c, d, e, f, 1f

This is to simulate the state of some other thread we’re loading in:

The program counter was at 13 (aka d), the registers were 1 through f (except b, which we need to be 4 for our infinite loop in the main code up top), and the flags were all on (1f in hex is 11111 in binary, signifying all flags on).

Then, just start running and enjoy as it executes a software interrupt at the end of each loop of our main program up top!

--------------------------------------------------------------------------------------

