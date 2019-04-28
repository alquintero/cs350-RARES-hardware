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

To implement interrupts, we let the hardware have most of the program control. Instead of checking when keyboards 1 and 2 had a character(s), we now had the keyboards automatically call the getchar+putchar function if they had a character. 

Issues we came across + fixed:

- **Resetting registers that hold key locations in memory.** We still had our infinite loop with busy waiting that would set R6 to 0x1000 and R8 to 0x2000 for the start of keyboard_1_handler and keyboard_2_handler, respectively. 
- **Branching commands conflict.** We also ran into an issue with conflicting branching commands (within the interrupt handler of keyboard 2) and so we had to manually change some register uses (i.e. not only use R11 but R12 and R13 to keep track of where we need to branch to in certain cases).
- **Disallowing interrupts within interrupts.** In order to disallow this (as if we allowed it this could lead to starvation), we only allowed an interrupt to be called if:
1. the keyboard had a character (avaiability bit was on)
2. the machine code was in the infinite loop (we weren't in another interrupt)
3. the R6 register for the keyboard_1_handler = 1000 or the R8 register for the keyboard_2_handler = 2000 (for the corresponding interrupt requested)
