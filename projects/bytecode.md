# Building a byte code interpreter

## Goal

Your goal in this project is to create a bytecode interpreter, a virtual machine (VM), that acts like the assembly code for an imaginary processor.

## Discussion

You will build a stack-based bytecode interpreter (or virtual machine) for a simple instruction set.  Instead of using registers like a register-based bytecode interpreter, a stack-based interpreter pushes temporary values onto an operand stack. For example, we would encode `3+4` as *push 3, push 4, add*. The *add* would pop to values off the stack, add them, and push the result of 7 onto the stack. 

Here's a complete program that prints 1234:

```
	ICONST 1234
	PRINT
	HALT
```

The `PRINT` instruction pops the top element off the operand stack and (as we will see later) it prints the element according to the type of tag of the top operand.

In order to load programs from the disk, I have defined a kind of assembly code format that specifies how many strings are used, where the functions are, and then the actual bytecodes. As loaded by [loader.c](https://github.com/USF-CS345-starterkits/parrt-bytecode/blob/master/src/loader.c) we have:

```
0 strings
1 functions maxaddr=0
	0: 4/main			// "main" has 4 characters
3 instr, 7 bytes
	ICONST 1234
	PRINT
	HALT
```

If we want to print a string literal, the assembly file format requires you to define the string and then reference it with `SCONST i` instruction. For example,

```
1 strings
   0: 5/hello
1 functions maxaddr=0\n"
	0: 4/main
3 instr, 5 bytes
	SCONST 0		// push a new String from strings[0]
	PRINT
	HALT
```

This program has a memory leak because `SCONST` creates a `String` `struct` from the `strings[0]` literal stored in the virtual machine by the loader. To free that we have to store it in a local variable and then manually free it before halting:

```
1 strings
    0: 5/hello
1 functions maxaddr=0\n"
    0: 4/main
7 instr, 17 bytes
    LOCALS 1    // make space for one local variable
    SCONST 0    // push a String from strings[0]
    STORE 0     // store the string in the first local variable
    LOAD 0      // push it back on the stack
    PRINT       // now we can finally print it
    SFREE 0     // ok, free the string at locals[0]
    HALT        // later, dude
```

You should look carefully through the unit tests [test_core.c](https://github.com/USF-CS345-starterkits/parrt-bytecode/blob/master/test/test_core.c) and [test_funcs.c](https://github.com/USF-CS345-starterkits/parrt-bytecode/blob/master/test/test_funcs.c). Unit tests are a bit of executable documentation that describes the operation of the virtual machine.

### Data types

The VM has three native types:

* integers (C type `int`)
* strings (C type struct `String`)
* boolean (C type `int`) (this type is only used internally; you cannot push true or false onto the stack manually, for example)

Local variables and the operand stack are tagged with type information. Each such "slot" therefore has two pieces of information: the type and the value. To allow the same memory cell to hold an `int`, `bool`, or `String *` we need to use a `union` in C:

```C
typedef struct {
	element_type type;
	union {
		int i;
		bool b;
		String *s;
	};
} element;
```

The operand stack is defined as:

```C
element stack[MAX_OPND_STACK];
```

and the local variable array within a function call activation record is defined as:

```C
element locals[MAX_LOCALS];
```

You should take a look at [vm.h](https://github.com/USF-CS345-starterkits/parrt-bytecode/blob/master/src/vm.h), which has all of these definitions created for you. You're welcome.

Instructions are stored in byte-addressable code memory and some of the instructions take operands from code memory rather than the operand stack. For example, *push 3* is encoded as `ICONST 3` where the `3` appears as a four byte integer directly after the byte representing the `ICONST` instruction. 

### Machine architecture

Our VM emulates the registers and memory structures of a real processor with fields of a C `struct`:

```C
typedef struct {
	// registers
	addr32 ip;        	// instruction pointer register
	int sp;             // stack pointer register
	int callsp;			// call stack pointer register

	byte *code;   		// byte-addressable code memory.
	int code_size;
	element stack[MAX_OPND_STACK]; 	// operand stack, grows upwards; word addressable
	Activation_Record call_stack[MAX_CALL_STACK];

	int num_functions;
	int max_func_addr;
	char **func_names;
	int num_strings;
	String **strings;

	char *trace;
	char *output;		// prints strcat on to the end of this buffer
} VM;
```

### Instruction set

| Instruction | Size | Operational Semantics |
|--------|--------|--------|
`HALT` | 1 | Stop the execution of the program
`IADD` | 1 | y := pop, x := pop, push x+y
`ISUB` | 1 | y := pop, x := pop, push x-y
`IMUL` | 1 |y := pop, x := pop, push x*y
`IDIV` | 1 |y := pop, x := pop, push x/y
`SADD` | 1 |t := pop, s := pop, push stringcat s, t
`OR` | 1 |y := pop, x := pop, push x or y
`AND` | 1 |y := pop, x := pop, push x and y
`INEG` | 1 |x := pop, push -x
`NOT` | 1 |x := pop, push not x
`I2S` | 1 |x := pop, push String.valueOf(x)
`IEQ` | 1 |y := pop, x := pop, push x==y
`INEQ` | 1 |y := pop, x := pop, push x!=y
`ILT` | 1 |y := pop, x := pop, push x<y
`ILE` | 1 |y := pop, x := pop, push x<=y
`IGT` | 1 |y := pop, x := pop, push x>y
`IGE` | 1 |y := pop, x := pop, push x>=y
`SEQ` | 1 |t := pop, s := pop, push strcmp(s,t)=0
`SNEQ` | 1 |t := pop, s := pop, push strcmp(s,t)!=0
`SGT` | 1 |t := pop, s := pop, push strcmp(s,t)>0
`SGE` | 1 |t := pop, s := pop, push s.equals(t)>=0
`SLT` | 1 |t := pop, s := pop, push s.equals(t)<0
`SLE` | 1 |t := pop, s := pop, push s.equals(t)<=0
`BR a` | 5 | ip := a
`BRF a` | 5 | b := pop, if b ip := a
`ICONST x` | 5 | push x
`SCONST i` | 3 | push strings[i]
`LOAD i` | 3 | push frame.locals[i]
`STORE i` | 3 | locals[i] = pop
`SINDEX` | 1 | i := pop, s := pop, push s[i]
`POP` | 1 | pop
`CALL a,n` | 7 | frame := new call stack frame<br>frame.nargs := n<br>frame.retaddr := ip + 1 + 4 + 2<br>push frame on call stack<br>for i=n-1..0:<br>&nbsp;&nbsp; frame.locals[i] := pop<br>frame.name = func_names[a]<br>ip := a
`LOCALS x` | 3 | frame.nlocals := x
`RET` | 1 | x := pop<br>frame := pop from call stack<br>ip := frame.retaddr<br>push x<br>ip := frame.retaddr
`PRINT` | 1 | x := pop, print x
`SLEN` | 1 | s := pop, push strlen(s)
`SFREE i` | 3 | free locals[i], locals[i] = NULL

You must advance the ip as part of the instruction decode.

The stack operations are: `push x` is `stack[++sp] := x` and `pop` is `stack[sp--]`.  The same is true for call stack.

### Executing instructions

The `void vm_exec(VM *vm, bool trace_to_stderr)` method represents your fetch-decode-execute loop. It executes until it runs out of instructions or hits a `HALT` instruction.

```
ip := look up "main" function address
while there is an instruction at code[ip]:
	fetch opcode at code[ip]
	decode with a switch on opcode
	execute the instruction with the `case`s
```

For your own debugging purposes and as part of the unit test, you need to emit tracing information. I have provided the functions for you, but you must call them appropriately. The trace should show the state of the operand and call stack after the execution of the current instruction. For example, given the program above that prints 1234, you should generate the following trace string:

```
0000:  ICONST         1234      calls=[ main=[ ] ]  stack=[ 1234 ] sp=0
0005:  PRINT                    calls=[ main=[ ] ]  stack=[ ] sp=-1
0006:  HALT                     calls=[ main=[ ] ]  stack=[ ] sp=-1
```

The first part is the address in decimal then instruction then any operands in code memory. After the instruction itself you see the call stack (which has any locals too) and then the operand stack state.

## Getting started

I have provided a [starter kit](https://github.com/USF-CS345-starterkits/parrt-bytecode) that you can pull into your repository. From the command line, clone your project repo and then pull in my starter kit:

```bash
$ git clone git@github.com:USF-CS345-S16/USERID-bytecode.git
$ cd USERID-bytecode/ # jump into dir containing YOUR empty repo
$ git pull git@github.com:USF-CS345-starterkits/parrt-bytecode.git master
```

You can get any updates to the starter kit by simply `git pull`ing again from `USF-CS345-starterkits/parrt-bytecode.git`.

## Building and testing

You might need to install `valgrind` and `cmake` on your system. If you are using a Mac, do this:

```bash
$ brew update
$ brew install valgrind
$ brew install cmake
```

I suggest that you use CLion, which knows how to deal with cmake builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows. It does not appear to use `valgrind` from within CLion.

From the command line, you can build and test all of your software as follows on UNIX:

```bash
$ cmake CMakeLists.txt
...
$ make
[ 16%] Built target c_unit
[ 50%] Built target vm
[ 66%] Built target test_core
[ 83%] Built target test_funcs
[100%] Built target wrun
$ make test
Running tests...
Test project /Users/parrt/courses/CS345/projects/bytecode
    Start 1: test_core
1/2 Test #1: test_core ........................   Passed    1.28 sec
    Start 2: test_funcs
2/2 Test #2: test_funcs .......................   Passed    1.14 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   2.42 sec
```

Using `ctest` directly (which is what CMake invokes), you can see all of the output for debugging purposes with `-V`:

```bash
$ ctest -V | more
...
1: ==98324== Memcheck, a memory error detector
1: ==98324== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
1: ==98324== Using Valgrind-3.11.0.SVN and LibVEX; rerun with -h for copyright info
1: ==98324== Command: ./test_core
1: ==98324== 
1: PASS hello
1: PASS locals
1: PASS sfree
1: PASS iadd
1: PASS isub_pos
1: PASS isub_neg
1: PASS imul
1: PASS idiv
1: PASS ieq
1: PASS icmp
1: PASS sadd
1: PASS i2s
1: PASS seq
1: PASS scmp
1: PASS sindex
1: PASS br
1: PASS brf
1: PASS while_stat
1: ==98324== 
1: ==98324== HEAP SUMMARY:
1: ==98324==     in use at exit: 34,909 bytes in 417 blocks
1: ==98324==   total heap usage: 733 allocs, 316 frees, 40,078,205 bytes allocated
1: ==98324== 
1: ==98324== LEAK SUMMARY:
1: ==98324==    definitely lost: 0 bytes in 0 blocks
1: ==98324==    indirectly lost: 0 bytes in 0 blocks
1: ==98324==      possibly lost: 0 bytes in 0 blocks
1: ==98324==    still reachable: 0 bytes in 0 blocks
1: ==98324==         suppressed: 34,909 bytes in 417 blocks
1: ==98324== 
1: ==98324== For counts of detected and suppressed errors, rerun with: -v
1: ==98324== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 16 from 16)
1/2 Test #1: test_core ........................   Passed    1.32 sec
...
```

## Deliverables

You must fill in the functions that are blank, but the primary one of course is the `vm_exec()` function. You must pass all unit tests, and of course `valgrind` should not find any memory leaks or memory errors.

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

