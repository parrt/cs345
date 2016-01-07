# Building a byte code interpreter

## Goal

Your goal in this project is to create a bytecode interpreter, a virtual machine (VM), that acts like the assembly code for an imaginary processor.

## Discussion

### Instruction set

| Instruction | Semantics |
|--------|--------|
`HALT` | Stop the execution of the program
`IADD` | y := pop, x := pop, push x+y
`ISUB` | y := pop, x := pop, push x-y
`IMUL` |y := pop, x := pop, push x*y
`IDIV` |y := pop, x := pop, push x/y
`SADD` |t := pop, s := pop, push stringcat s, t
`OR` |y := pop, x := pop, push x or y
`AND` |y := pop, x := pop, push x and y
`INEG` |x := pop, push -x
`NOT` |x := pop, push not x
`I2S` |x := pop, push String.valueOf(x)
`IEQ` |y := pop, x := pop, push x==y
`INEQ` |y := pop, x := pop, push x!=y
`ILT` |y := pop, x := pop, push x<y
`ILE` |y := pop, x := pop, push x<=y
`IGT` |y := pop, x := pop, push x>y
`IGE` |y := pop, x := pop, push x>=y
`SEQ` |t := pop, s := pop, push strcmp(s,t)=0
`SNEQ` |t := pop, s := pop, push strcmp(s,t)!=0
`SGT` |t := pop, s := pop, push strcmp(s,t)>0
`SGE` |t := pop, s := pop, push s.equals(t)>=0
`SLT` |t := pop, s := pop, push s.equals(t)<0
`SLE` |t := pop, s := pop, push s.equals(t)<=0
`BR a` | ip := a
`BRF a` | b := pop, if b ip := a
`ICONST x` | push x
`SCONST i` | push strings[i]
`LOAD i` | push frame.locals[i]
`STORE i` | locals[i] = pop
`SINDEX` | i := pop, s := pop, push s[i]
`POP` | pop
`CALL a,n` | frame := new call stack frame<br>frame.nargs := n<br>frame.retaddr := ip + 1 + 4 + 2<br>push frame on call stack<br>for i=n-1..0:<br>&nbsp;&nbsp; frame.locals[i] := pop<br>ip := a
`LOCALS x` | frame.nlocals := x
`RET` | x := pop, ip := frame.retaddr
`PRINT` |
`SLEN` |
`SFREE i` |

The stack operations are: `push x` is `stack[++sp] := x` and `pop` is `stack[sp--]`.  The same is true for call stack.

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

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

