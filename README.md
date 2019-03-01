# cs350-RARES-hardware
This is for our hardware team.

## DESIGN DOC

*For 3/3/18 Due Date.*
Our original idea for the hardware was to create a currently unused opcode for our putchar function and process it through the ALSU similarly to previously created functions in CS240. 

Two parameters: 
Eventually, we would take in input through the first keyboard but for now we are manually inputting input. The ASCII value of that character will go into R2. For string input, we are doing this character by character in accordance with the putchar function written by the software team (as opposed to taking in the string all at once). The ASCII value of each character, having been put into R1, would go into our ROM via HAssem and be taken in by our ALSU along with the opcode for putchar. The putchar circuit within the ALSU would simply return that same ASCII value and output it. From the ALSU, the ASCII value would go into R2. Then, R1 would hold our first terminal as input. 

As we worked with our microprocessor made last semester, we discovered there was much to do in order to make putchar possible. We also had to create Call and Return as our microprocessor did not currently have it. Then, we adjusted our original concept to make sure everything could flow in the correct paths and ultimately output into the terminal correctly. 
