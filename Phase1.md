# The phase1 for Attack Lab

## Goal
Since we're using buffer overflow, the function we will attack is the one who cal Gets(buf), which is the <*getbuf*> funtion. Specifically, we need to change the return address for <*getbuf*>.

## Phase 1
For phase1, we only need to change the return address of <*getbuf*>, redirecting it to another function <*touch1*>. Therefore, we need to know how many bytes to write.
The Stack structure is as follows:


|    Stack    |
|-------------|
|  Ret Addr   |
| ...|
| ...|  // denote 8 bytes
| ...|  %rdi = %rsp 

%rdi for Gets is actually %rsp, so the string input will start from rsp and then grows up. Since the stack frame of <*getbuf*> function is 0x18 (4bytes), we need first fill in the stack with 3 bytes and then modify the Return Address to be the start address of touch1


## Phase 2
Phase2: inject some codes on Stack and run them.

#### Important Notes:
1. String input will be loaded to stack from lower address to higher address, i.e, the first character has lowest address on stack.
2. If the input is interpreted as **instructions**, then it will be read from lower address to higher address; if interpreted as **address** (or other integer value), it will be read from higher address to lower ones because of Little Endian. NOTE: if using pop %rsi, data will be stored from lower to higher address. e.g. to store 1234 in %rsi, we need:

| Stack|
|------|
|  34  |
|  12  |  // denote single bytes

3. Ret is essentially pop %rip. To execute codes on stack, we need to pass the address of codes on stack to %rip, which is done by pop. In other words, store begining address of stack-instructions at %rsp.


## Phase 3
Phase3 requires to get the address of a string, and put it into %rdi.

Since for phase2 and phase3 the stack doesn't use ranomization we can determine where to inject the string, what its address will be, and where to inject exploit code. 
**Need to compute stack location carefully, because function call might override our data**

### A Sum for phase1, phase2 and phase3
1. Stack is not randomized, so we can compute the location of stack, and directly put static address on the stack.
2. Code on stack is executable, and thus we can write our own codes.

## Phase 4
Phase 4 and phase 5 are about ROP. Two important features:
1. Can't put static address anymore(because of randomization of stack)
2. Stack is marked as non-executable.

Phase 4 is easier than phase5, because we only need to pop stuff to register. 

## Phase 5
Phase 5 is the most difficult one. Several take-aways here:
1. Based on the given farm, we don't have the ability to write directly into memory so the string in memory has to be set by buffer overflow.
2. Then, the key point is how to get the address of the string. Given that the stack is randomized, we can only use **relative address** so one idea is to use stack pointer %rsp to track the location.
3. But if we put the string onto stack, we won't be able to proceed, i.e. can't use **ret** anymore (ret to an unknown address represented by the string bytes).
4. It seems that pop %rdi will solve the problem (by skipping the the string). But since there's no pop following mov %rsp %rax, doing pop will prevent us from getting the address of the string.
5. So, the thing I didn't notice was that we actually have an arithmetic operation lea (%rdi, %rsi, 1) %rax. Based on this arithmetic operation, we can put the string away from %rsp, and getting its location by %rsp + offset, as shown in the figure below.

|Stack|
|-----|
|String|
|Return Address of touch3|
|...|
|...|

So actually the %rsp won't pass the string. 
