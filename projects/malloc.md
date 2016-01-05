# Implementating malloc/free

## Goal

Dynamic memory allocation is an important part of a programming language's runtime support. You goal in this project is to create an implantation of the standard C library functions `malloc()` and `free()`. There are many different implementation strategies as we discussed in class, but we'll be building the simplest *free list* mechanism. Recall in the strategy that all free chunks of memory are linked together as part of a singly linked list. New memory is allocated by picking the first free chunk that fits, splitting it into two chunks if it's not an exact fit. Free'd memory is inserted at the head of the free list. The unit tests assume this specific allocation policy so we can check the physical structure of your free list for accuracy. Initially, memory to hand out from `malloc()` is obtained from the operating system using a function I provide to you called `morecore()`.

The primary learning objective is to get comfortable with language-implementation level C coding: "bit twiddling", pointer pinball (heh, just coined that term and I like it), and overlaying `struct`s on top of different memory chunks.

## Discussion

At its most abstract, a free list `malloc`/`free` is nothing more than searching and adding elements to a singly-linked list. The only tricky bits are related to the header information we need to store along with each chunk we allocate so that we can deallocate it later.

| n | size with header|
|--|-----------------|
| 0| 10 |



Here are the relevant chunks from [malloc.c](https://github.com/USF-CS345-starterkits/parrt-malloc/blob/master/src/malloc.c) that you have to fill in:

```C
void heap_init(size_t max_heap_size) {
	// TODO: if heap already allocated, call heap_shutdown() first
	// TODO: record the size requested and ask morecore() for that much memory
	// TODO: initialize the linked list head variable: freelist
}

void heap_shutdown() {
	// TODO: call dropcore(), reset the heap and heap size variables
}

/** Allocate and returned a chunk of memory at least size bytes long.
 *  Split a bigger chunk if no exact fit, setting size of split chunk returned.
 *  size arg must be big enough to hold Free_Header and padded to 4 or 8
 *  bytes depending on word size.
 */
void *malloc(size_t size) {
	// TODO: if keep not allocated, initialized with the default heap size
	// TODO: return NULL if we are out of memory
	// TODO: add enough space so that we have room for our header
	// TODO: aligned to a word boundary
	// TODO: find the first free chunk that's big enough to hold the requested chunk and our header
	// TODO: mark this chunk as busy and return it
}

/* Free chunk p by adding to head of free list */
void free(void *p) {
	// TODO: return if p is NULL or out of range of the heap
	// TODO: return if p is already free
	// TODO: add to the head of the free list
}
```

There is also an important function the returns information about the status of the heap:

```C
/* Walk heap jumping by size field of chunk header. Return an info record. */
Heap_Info get_heap_info() {
	// TODO: while p is in bounds of the heap, walk heap jumping by chunksize(p)
	// TODO: 	count busy and free chunk sizes
	// TODO: return a Heap_Info with relevant data
}
```

It is critical to unit testing.

## Getting started

I have provided a [starter kit](https://github.com/USF-CS345-starterkits/parrt-malloc) that you can pull into your repository. From the command line, clone your project repo and then pull in my starter kit:

```bash
$ git clone git@github.com:USF-CS345-S16/USERID-malloc.git
$ cd USERID-malloc/ # jump into dir containing YOUR empty repo
$ git checkout -b master # it's empty so we must create a master manually
Switched to a new branch 'master'
$ git remote add starterkit git@github.com:USF-CS345-starterkits/parrt-malloc.git
$ git remote -v # show me what remote repositories are associated with mine
origin	git@github.com:USF-CS345-S16/parrt-malloc.git (fetch)
origin	git@github.com:USF-CS345-S16/parrt-malloc.git (push)
starterkit	git@github.com:USF-CS345-starterkits/parrt-malloc.git (fetch)
starterkit	git@github.com:USF-CS345-starterkits/parrt-malloc.git (push)
$ git pull starterkit master
remote: Counting objects: 25, done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 25 (delta 3), reused 20 (delta 3), pack-reused 0
Unpacking objects: 100% (25/25), done.
From github.com:USF-CS345-starterkits/parrt-malloc
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> starterkit/master
$ git branch -v
* master 6eba2cc initial add
$ git push origin master # push all that stuff back to your repo
```

You can get any updates to the starter kit by simply `git pull`ing again from `starterkit `, which will merge in any changes.

## Building and testing

I suggest that you use CLion, which knows how to deal with cmake builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows. It does not appear to use `valgrind` from within CLion.

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

Every time you commit to your repository, your software will automatically be downloaded and tested on the Travis continuous integration server using maven. Check out [https://travis-ci.com/USF-CS345-S16/USERID-malloc](https://travis-ci.com/USF-CS345-S16/USERID-malloc) where USERID is your github user id. You will not be able to see the repositories of other students.

## Deliverables

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

