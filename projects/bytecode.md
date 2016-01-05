# Building a byte code interpreter
## Goal

## Discussion

## Getting started

I have provided a [starter kit](https://github.com/USF-CS345-starterkits/parrt-bytecode) that you can pull into your repository. From the command line, it looks like:

```bash
$ cd userid-bytecode # jump into dir containing your empty repo
$ git pull git@github.com:USF-CS345-starterkits/parrt-bytecode.git master
```

## Building and testing

I suggest that you use CLion, which knows how to deal with cmake builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows. It does not appear to use `valgrind` from within CLion.

From the command line, you can build and test all of your software as follows on UNIX:

```bash
$ cmake CMakeLists.txt 
...
$ make
[ 30%] Built target c_unit
[ 60%] Built target malloc
[ 80%] Built target test_basic_sanity
[100%] Built target test_replay
$ make test
Running tests...
Test project /Users/parrt/courses/CS345/projects/malloc
    Start 1: test_basic_sanity
1/3 Test #1: test_basic_sanity ................   Passed    0.97 sec
    Start 2: test_replay_no_valgrind
2/3 Test #2: test_replay_no_valgrind ..........   Passed    3.86 sec
    Start 3: test_replay
3/3 Test #3: test_replay ......................   Passed   36.01 sec

100% tests passed, 0 tests failed out of 3

Total Test time (real) =  40.84 sec
```

Using `ctest` directly (which is what CMake invokes), you can see all of the output for debugging purposes with `-V`:

```bash
$ ctest -V | more
...
1: ==98207== Memcheck, a memory error detector
1: ==98207== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
1: ==98207== Using Valgrind-3.11.0.SVN and LibVEX; rerun with -h for copyright info
1: ==98207== Command: ./test_basic_sanity
1: ==98207== 
1: PASS malloc0
1: PASS malloc1
1: PASS malloc_word_size
1: PASS malloc_2x_word_size
1: PASS one_malloc
1: PASS two_malloc
1: PASS free_NULL
1: PASS free_random
1: PASS free_stale
1: PASS malloc_then_free
1: ==98207== 
1: ==98207== HEAP SUMMARY:
1: ==98207==     in use at exit: 34,941 bytes in 418 blocks
1: ==98207==   total heap usage: 518 allocs, 100 frees, 41,917 bytes allocated
1: ==98207== 
1: ==98207== LEAK SUMMARY:
1: ==98207==    definitely lost: 0 bytes in 0 blocks
1: ==98207==    indirectly lost: 0 bytes in 0 blocks
1: ==98207==      possibly lost: 0 bytes in 0 blocks
1: ==98207==    still reachable: 0 bytes in 0 blocks
1: ==98207==         suppressed: 34,941 bytes in 418 blocks
1: ==98207== 
1: ==98207== For counts of detected and suppressed errors, rerun with: -v
1: ==98207== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 17 from 17)
1/3 Test #1: test_basic_sanity ................   Passed    0.95 sec
...
```

## Deliverables

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

