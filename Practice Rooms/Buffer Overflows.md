
When a program runs on a machine, the computer runs the program as a process. Current computer architecture allows multiple processes to be run concurrently(at the same time by a computer). While these processes may appear to run at the same time, the computer actually switches between the processes very quickly and makes it look like they are running at the same time. Switching between processes is called a context switch. Since each process may need different information to run(e.g. The current instruction to execute), the operating system has to keep track of all the information in a process. The memory in the process is organised sequentially and has the following layout:

![](https://i.imgur.com/KmWsaIs.png)

- User stack contains the information required to run the program. This information would include the current program counter, saved registers and more information. The section after the user stack is unused memory and it is used in case the stack grows(downwards)
    
- Shared library regions are used to either statically/dynamically link libraries that are used by the program
    
- The heap increases and decreases dynamically depending on whether a program dynamically assigns memory. Notice there is a section that is unassigned above the heap which is used in the event that the size of the heap increases.
    
- The program code and data stores the program executable and initialised variables.


##### x86-64 Procedures

  

A program would usually comprise of multiple functions and there needs to be a way of tracking which function has been called, and which data is passed from one function to another. The stack is a region of contiguous memory addresses and it is used to make it easy to transfer control and data between functions. The top of the stack is at the lowest memory address and the stack grows towards lower memory addresses. The most common operations of the stack are:

  
**Pushing**: used to add data onto the stack

**Popping**: used to remove data from the stack

![[Pasted image 20240407004254.png]]


push var

This is the assembly instruction to push a value onto the stack. It does the following:

- Uses var or value stored in memory location of var
    

![](https://i.imgur.com/TFH7KDf.png)  

- Decrements the stack pointer(known as `rsp`) by 8
    
- Writes above value to new location of `rsp`, which is now the top of the stack
    

![](https://i.imgur.com/fLz86wR.png)  

`pop var`

This is an assembly instruction to read a value and pop it off the stack. It does the following:

- Reads the value at the address given by the stack pointer
    

Stack Bottom()

![](https://i.imgur.com/tuH0By9.png)  

Stack Top(memory location 0x0)(`rsp` points here)

- Store the value that was read from `rsp` into var
    
- Increment the stack pointer by 8
    

![](https://i.imgur.com/dWPAXEF.png)



It’s important to note that the memory does not change when popping values of the stack - it is only the value of the stack pointer that changes!   

Each compiled program may include multiple functions, where each function would need to store local variables, arguments passed to the function and more. To make this easy to manage, each function has its own separate stack frame, where each new stack frame is allocated when a function is called, and deallocated when the function is complete.   

![](https://i.imgur.com/0OsNBwQ.png)

Example program 

```c
int add(int a, int b){
   int new = a + b;
   return new;
}

int calc(int a, int b){
   int final = add(a, b);
   return final;
}

calc(4, 5)
```

The explanation assumes that the current point of execution is inside the calc function. In this case calc is known as the caller function and add is known as the callee function. The following presents the assembly code inside the calc function

![](https://lh3.googleusercontent.com/drfkuTdzr6yTKq2tPgCO95_knSqbNa0_iApkl3w62yDGZIOu_6ZxMMcQE4SD-X4p-3AW656Azf3iWqCjRqC4S6O8PbcseQS3r0mzuyB0T2WZdfxs7HtVVqGheU5R2zBVSjEix3sS)

![](https://i.imgur.com/14sqeAZ.png)


The add function is invoked using the call operand in assembly, in this case callq sym.add. The call operand can either take a label as an argument(e.g. A function name), or it can take a memory address as an offset to the location of the start of the function in the form of call *value. Once the add function is invoked(and after it is completed), the program would need to know what point to continue in the program. To do this, the computer pushes the address of the next instruction onto the stack, in this case the address of the instruction on the line that contains movl %eax, local_4h. After this, the program would allocate a stack frame for the new function, change the current instruction pointer to the first instruction in the function, change the stack pointer(rsp) to the top of the stack, and change the frame pointer(rbp) to point to the start of the new frame.   

![](https://lh3.googleusercontent.com/IgWTFeegf8jSIlpJAz-jdJW_I477jiXWvY-kkGUjZ_iIIGgj_gBKac0-bzonwNrlpAw6sUvh7ZkejC50peTnWKfxZAUUiQA_QiYwlAkgDA7gkOdJZIqpDruAeVeqOODCEfCsv325)  

  
![](https://i.imgur.com/BZMUlMe.png)

Once the function is finished executing, it will call the return instruction(retq). This instruction will pop the value of the return address of the stack, deallocate the stack frame for the add function, change the instruction pointer to the value of the return address, change the stack pointer(rsp) to the top of the stack and change the frame pointer(rbp) to the stack frame of calc.

  
![](https://i.imgur.com/VF6LfjU.png)


Now that we’ve understood how control is transferred through functions, let’s look at how data is transferred.   

In the above example, we save that functions take arguments. The calc function takes 2 arguments(a and b). Upto 6 arguments for functions can be stored in the following registers:  

- rdi
    
- rsi
    
- rdx
    
- rcx
    
- r8
    
- r9
    

***Note: rax is a special register that stores the return values of the functions(if any).***

If a function has anymore arguments, these arguments would be stored on the functions stack frame.   

We can now see that a caller function may save values in their registers, but what happens if a callee function also wants to save values in the registers? To ensure the values are not overwritten, the callee values first save the values of the registers on their stack frame, use the registers and then load the values back into the registers. The caller function can also save values on the caller function frame to prevent the values from being overwritten. Here are some rules around which registers are caller and callee saved:  

- rax is caller saved
    
- rdi, rsi, rdx, rcx r8 and r9 are called saved(and they are usually arguments for functions)
    
- r10, r11 are caller saved
    
- rbx, r12, r13, r14 are callee saved 
    
- rbp is also callee saved(and can be optionally used as a frame pointer)
    
- rsp is callee saved
    

So far, this is a more thorough example of the run time stack:

|   |
|---|
|![](https://i.imgur.com/vA0ug3J.png)|


---

##### Endianess

In the above programs, you can see that the binary information is represented in hexadecimal format. Different architectures actually represent the same hexadecimal number in different ways, and this is what is referred to as Endianess. Let’s take the value of 0x12345678 as an example. Here the least significant value is the right most value(78) while the most significant value is the left most value(12).

Little Endian is where the value is arranged from the least significant byte to the most significant byte:  

![](https://i.imgur.com/tSYo8AS.png)  

Big Endian is where the value is arranged from the most significant byte to the least significant byte.

![](https://i.imgur.com/ltUjHQ7.png)  

Here, each “value” requires at least a byte to represent, as part of a multi-byte object.

---

##### Overwriting Variables

With how the compiler and stack are configured, when variables are allocated, they would need to be aligned to particular size boundaries(e.g. 8 bytes, 16 byte) to make it easier for memory allocation/deallocation. So if a 12 byte array is allocated where the stack is aligned for 16 bytes this is what the memory would look like:  

![](https://i.imgur.com/kjxX8SC.png)

