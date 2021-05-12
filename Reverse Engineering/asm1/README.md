# asm1
AUTHOR: SANJAY C
## Problem
> What does asm1(0x8be) return? Submit the flag as a hexadecimal value (starting with '0x'). [Source](https://github.com/Henry1601/PicoCTF-Writeup/blob/6eecbc6c032866c91e5e09daebb27ac1482603a7/Reverse%20Engineering/asm1/test1.S)
## Solution
To me, there are two ways to solve assembly problems like this one. First and recommend way is to read it line by line so we could understand it better. Second is to make it executable and let computer do it, we would get the result faster and more convenient. Here I will explain both but the second way will only be explained in this post.

Note that this will require basic knowledge of assembly code that I can't cover it all so you can look up on the Internet.

Let's begin! Here is the code
```
asm1:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	cmp    DWORD PTR [ebp+0x8],0x71c
	<+10>:	jg     0x512 <asm1+37>
	<+12>:	cmp    DWORD PTR [ebp+0x8],0x6cf
	<+19>:	jne    0x50a <asm1+29>
	<+21>:	mov    eax,DWORD PTR [ebp+0x8]
	<+24>:	add    eax,0x3
	<+27>:	jmp    0x529 <asm1+60>
	<+29>:	mov    eax,DWORD PTR [ebp+0x8]
	<+32>:	sub    eax,0x3
	<+35>:	jmp    0x529 <asm1+60>
	<+37>:	cmp    DWORD PTR [ebp+0x8],0x8be
	<+44>:	jne    0x523 <asm1+54>
	<+46>:	mov    eax,DWORD PTR [ebp+0x8]
	<+49>:	sub    eax,0x3
	<+52>:	jmp    0x529 <asm1+60>
	<+54>:	mov    eax,DWORD PTR [ebp+0x8]
	<+57>:	add    eax,0x3
	<+60>:	pop    ebp
	<+61>:	ret
```
I'll break it into pieces and explain each one.
```
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
```
We know that we are putting 0x8be (input value of function) to stack, which gets pushed into **EBP** and then moved into **ESP** on line 0 and 1.

```
	     Stack
	|-------------|		(high memory)
	|----0x8be----|		<--- ebp + 0x8 (input value)
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|-----ebp-----|		<--- ebp
	|-------------|		(low memory)
```
```
	<+3>:	cmp    DWORD PTR [ebp+0x8],0x71c
	<+10>:	jg     0x512 <asm1+37>
```
In the next 2 line we are seeing that we are comparing 0x8be (value at [ebp+0x8]) with 0x71c and since 0x8be is greater than 0x71c, we take the jump to asm1+37. The `jg` means "**jump if greater**". Now we are going to line 37.
```
	<+37>:	cmp    DWORD PTR [ebp+0x8],0x8be
	<+44>:	jne    0x523 <asm1+54>
	<+46>:	mov    eax,DWORD PTR [ebp+0x8]
	<+49>:	sub    eax,0x3
	<+52>:	jmp    0x529 <asm1+60>
```
Here we are comparing input value with 0x8be. The `jne` means "**jump if not equal**" and obviously 0x8be is equal 0x8be so we do not take the jump. In line 46 and 49, we subtract 0x3 from 0x8be then move to EAX, which would be 0x8bb now. We then unconditionally jump `jmp` to line 60.
```
	<+60>:	pop    ebp
	<+61>:	ret
```
At line 60, the stack is popped and EAX is returned. Since EAX is equal to 0x8bb, that is our flag.

Here is the function re-written in C code:
```
	int asm1(int val) {
		if(val > 0x71c)
			return val - 0x3;
		else if(val != 0x6cf)
			return val - 0x3;
		else
			return val + 0x3;
	}
```
## Second solution
Now I will make use of computer. First, I rewrite the code a little bit:
```
	.intel_syntax noprefix
	.global asm1

	asm1:
	    push   ebp
	        mov    ebp,esp
	        cmp    DWORD PTR [ebp+0x8],0x71c
	        jg     b37
	        cmp    DWORD PTR [ebp+0x8],0x6cf
	        jne    b29
	        mov    eax,DWORD PTR [ebp+0x8]
	        add    eax,0x3
	        jmp    b60
	b29:
	        mov    eax,DWORD PTR [ebp+0x8]
	        sub    eax,0x3
	        jmp    b60
	b37:
	        cmp    DWORD PTR [ebp+0x8],0x8be
	        jne    b54
	        mov    eax,DWORD PTR [ebp+0x8]
	        sub    eax,0x3
	        jmp    b60
	b54:
	        mov    eax,DWORD PTR [ebp+0x8]
	        add    eax,0x3
	b60:
	        pop    ebp
	        ret
```
*Note that `.intel_syntax noprefix` is used for GCC.*

We then, build a C program to run it:
```
	#include<stdio.h>

	int main() {
	    printf("Flag is: 0x%x\n", asm1(0x8be));
	    return 0;
	}
```
Now, use GCC to compile 2 files (I use Linux)
```
	gcc -m32 -c asm1.S -o asm1.o		(compile assembly file to object)
	gcc -m32 -w -c run.c -o run.o		(compile C file to object)
	gcc -m32 run.o asm1.o -o result		(place output to result file)
```
*Note that GCC automatically compile 64-bit objects by default so we use `-m32` parameter to compile 32-bit objects.*

Finally, run result file (`./result`) and we get the flag. But as I said before, it's recommend to do with first way since it helps you to understand the code better. Computer-way is should only use incase lack of time.

That's it. *Have fun hacking!*
## Flag
`0x8bb`
