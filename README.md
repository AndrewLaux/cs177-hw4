# Task 1
Changing the control flow of the program by modification of the return address of the authenticate function. 
In this case overflow occured when the executable was provided with an argument longer than 21 bytes 
(as indicated by the instruction lea 0x15($ebp), $eax). An addition 4 bytes of overflow was needed to write past the memory 
for the saved base pointer. The return adress in question was overwitten with 0x0804867d, just past the jne instruction 
responsible for pass/fail control logic.

# Task 2
In this task we want to exploit the code in auth so that it spawns a shell with escalated priveleges. This is accomplished using 
shell injection. The compiler in this case reserves 92 bytes for a string buffer in the authenticate function plus 4 more bytes 
for the stack frame pointer. By overflowing all 96 bytes we are able to overwrite the return address of the function so that it 
jumps back into the 96 bytes buffer. We want to jump back into the buffer because, in this case, it is large enough to hold our 
45 byte shellcode. I had two major problems with this task: firstly I was taking for granted alephone's claim that his shellcode 
contained 46 bytes, this was offsetting my jump pointer by one byte preventing me from executing my injected code. I solved 
this by using len(shellcode) to precisely determine its length. The other major challenge was anticipating the absolute address 
on the stack of my code. Becoming familiar with the options of gdb helped me choose my jump address. I set a breakpoint right 
after the strcpy function and use x/40w $esp to get a good look at the stack after the program takes my input. I also make sure 
to run GDB with an input of equivalent (100 bytes) length because the amount of input given to the program seems to influence 
addressing. In addition targeting the shell code is made easier via implementation of a NOP sled. I prepend my shell code with NOPs 
so landing anywhere in NOPs will "slide" into my shellcode.

# Task 3
In this task the program does not set aside adequite room for shellcode in the buffer. The solution is to define an enviroment 
variable. The python function os.execv allows for passing in of environment variables, however, I wanted to check the addressing in 
gdb given a large environment varible so I opted to export to SHELLCODE in the virtual machine. I ran into problems trying to export 
byte values to the environment variable. The solution was define python code within the export command and resolve it as input for the 
export function. eg `$export SHELLCODE=$(python -c 'print "\x90"')` etc I also located the location of the env variable in gdb 
using `x/s *((char**)environ+1)` Then the input for this task is similar to that of task 1 except the target is an environment variable 
on the stack.

# Task 4
This time code was compiled with with stack protection enabled. This prevents the execution of code on the stack which meant 
the buffer overflow schemes used in tasks 1-3 were no longer of use. The first big hint I got when I ran the shell program from 
`./my_program shell` The message about descalting priveleges with the :D got me thinking. On analyzing shell with gdb I dound that, 
indeed, part of shells function was to set effective group ID. Perhaps a different shell will work? I tried running the shell from my 
home dir instead of shell in MrCode using my_programs and sure enough, It worked. Mostly thanks to the professpors hint but also, in 
hindsight, I think the sticky bit in the priveleges for `my_programs` was also a big giveaway. 
