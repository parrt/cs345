# Implementating malloc/free

## Goal

Dynamic memory allocation is an important part of a programming language's runtime support. You goal in this project is to create an implantation of the standard C library functions `malloc()` and `free()`. There are many different implementation strategies as we discussed in class, but we'll be building the simplest *free list* mechanism. Recall in the strategy that all free chunks of memory are linked together as part of a singly linked list. New memory is allocated by picking the first free chunk that fits, splitting it into two chunks if it's not an exact fit. Free'd memory is inserted at the head of the free list. The unit tests assume this specific allocation policy so we can check the physical structure of your free list for accuracy. Initially, memory to hand out from `malloc()` is obtained from the operating system using a function I provide to you called `morecore()`.

The primary learning objective is to get comfortable with language-implementation level C coding: "bit twiddling", pointer pinball (heh, just coined that term and I like it), and overlaying `struct`s on top of different memory chunks.

## Discussion

At its most abstract, a free list `malloc`/`free` is nothing more than searching and adding elements to a singly-linked list. The only tricky bits are related to the header information we need to store along with each chunk we allocate so that we can deallocate it later.

### Chunk headers

To allocate a chunk of, say, 10 bytes, we actually need to allocate extra space to keep track of its size so the chunk size is really 10 + sizeof(int). We represent a chunk as a `struct` in C (see [malloc.h](https://github.com/USF-CS345-starterkits/parrt-malloc/blob/master/src/malloc.h)):

```C
typedef struct _Busy_Header {
	uint32_t size;         // 31 bits for size and 1 bit for inuse/free; includes header data
	unsigned char mem[]; // nothing allocated; just a label to location after size
} Busy_Header;
```

The `struct` allows us to indicate that we want a 32-bit (`uint32_t`) integer stored at the start of any chunk allocated via our `malloc()`. Given an arbitrary pointer, `p`, from our heap, expression `p->size` tells us how big that chunk is.

Because we need to add these chunks to a linked list, we actually need the usual `next` pointer stored in the header as well.

```C
typedef struct _Free_Header {
	uint32_t size;
	struct _Free_Header *next; // lives inside user data area when free but not when in use
} Free_Header;
```

So, a free header is bigger than a busy (allocated) header. That's why you see our minimum size as:

```C
static const size_t MIN_CHUNK_SIZE = sizeof(Free_Header);
```

When we `free()` a chunk, `p`, we need to access that `p->next` field to link it into the free list. A chunk of requested size 0 would still need to allocate the minimum header size, `sizeof(Free_Header)`. All chunks are at least the size.

We don't want to waste space however so we don't just add `sizeof(Free_Header)` to the requested size, say, `n`. First, we tried to get away with `n + sizeof(Busy_Header)`, which is smaller than `n + sizeof(Free_Header)`. If `n + sizeof(Busy_Header)` and smaller than the minimum chunk size, then we create a chunk of the minimum size, `sizeof(Free_Header)`. This is codified in the following function.

```C
/* Pad size n to include header */
static inline size_t size_with_header(size_t n) {
	return n + sizeof(Busy_Header) <= MIN_CHUNK_SIZE ? MIN_CHUNK_SIZE : n + sizeof(Busy_Header);
}
```

For efficiency reasons or even to make things work out right depending on the underlying memory architecture, we need to return a pointer from `malloc` that is word aligned. For a 64-bit word, as most of our machines have today, that means we need to return a pointer ending with 3 binary zeroes: `0b....000`.  Because we're going to return `&p->mem` for chunk `p` from `malloc`, we need to make sure that our header is padded so `mem` starts on a word boundary. Here is the alignment kung fu:

```C
/* Align n to nearest word size boundary (4 or 8) */
static inline size_t align_to_word_boundary(size_t n) {
	return (n & ALIGN_MASK) == 0 ? n : (n + WORD_SIZE_IN_BYTES) & ~ALIGN_MASK;
}
```

With a word size of 8 (64 bits), we use a bitmask of 7 to examine the lowest three binary digits.  If we `&` (bitwise AND) `n` with 7 and get 0, that means there were no bits and `n` is already word aligned. If not, we can add 8 bytes to `n` and then mask off the lowest three bits by ANDing with `~0x7` or `~0b333`. The following table makes it easier to visualize what's going on. For any given pointer, `p`, we first add 8 and then mask. The value in the right column is the word aligned pointer value. This was computed with [showsizes.c](https://github.com/USF-CS345-starterkits/parrt-malloc/blob/master/src/showsizes.c) from your starter kit.

| n | n&0x7 | n+8| (n+8) & ~0x7 | (n&0x7)==0 ? n : (n+8) & ~0x7|
|---|-------|----|--------------|----------------------------|
| 0 | 0 | 8 | 8 | 0 |
| 1 | 1 | 9 | 8 | 8 |
| 2 | 2 | 10 | 8 | 8 |
| 3 | 3 | 11 | 8 | 8 |
| 4 | 4 | 12 | 8 | 8 |
| 5 | 5 | 13 | 8 | 8 |
| 6 | 6 | 14 | 8 | 8 |
| 7 | 7 | 15 | 8 | 8 |
| 8 | 0 | 16 | 16 | 8 |
| 9 | 1 | 17 | 16 | 16 |
| 10 | 2 | 18 | 16 | 16 |
| 11 | 3 | 19 | 16 | 16 |
| 12 | 4 | 20 | 16 | 16 |
| 13 | 5 | 21 | 16 | 16 |
| 14 | 6 | 22 | 16 | 16 |
| 15 | 7 | 23 | 16 | 16 |
| 16 | 0 | 24 | 24 | 16 |
| 17 | 1 | 25 | 24 | 24 |
| 18 | 2 | 26 | 24 | 24 |
| 19 | 3 | 27 | 24 | 24 |
| 20 | 4 | 28 | 24 | 24 |
| 21 | 5 | 29 | 24 | 24 |
| 22 | 6 | 30 | 24 | 24 |
| 23 | 7 | 31 | 24 | 24 |
| 24 | 0 | 32 | 32 | 24 |
| 25 | 1 | 33 | 32 | 32 |
| 26 | 2 | 34 | 32 | 32 |
| 27 | 3 | 35 | 32 | 32 |
| 28 | 4 | 36 | 32 | 32 |
| 29 | 5 | 37 | 32 | 32 |
| 30 | 6 | 38 | 32 | 32 |
| 31 | 7 | 39 | 32 | 32 |
| 32 | 0 | 40 | 40 | 32 |

The showsizes program also dumps some other useful information:

```
int size 4
pointer size 8
sizeof(Busy_Header) == 4
sizeof(Free_Header) == 16
BUSY_BIT == 80000000
SIZEMASK == 7fffffff
```

And then provides a table that is more relevant to our problem. Given a requested size, `n`, what is the size of the chunk we need, and then what is the word aligned version of that (function `request2size()`):

| n | size with header | aligned to word boundary |
|---|------------------|---------------|
| 0 | 16 | 16 |
| 1 | 16 | 16 |
| 2 | 16 | 16 |
| 3 | 16 | 16 |
| 4 | 16 | 16 |
|...| ...| ... |
| 11 | 16 | 16 |
| 12 | 16 | 16 |
| 13 | 17 | 24 |
| 14 | 18 | 24 |
|...| ...| ... |
| 19 | 23 | 24 |
| 20 | 24 | 24 |
| 21 | 25 | 32 |
| 22 | 26 | 32 |
| 23 | 27 | 32 |
|...| ...| ... |
| 27 | 31 | 32 |
| 28 | 32 | 32 |
| 29 | 33 | 40 |
| 30 | 34 | 40 |
| 31 | 35 | 40 |
| 32 | 36 | 40 |

### What you have to fill in

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

Getting that information correct is critical to unit testing.

### How to test memory allocation

Aside from a few obvious manual tests that call your memory allocation and free routines, what's a good way to test your functions in a realistic environment? Well, one way is to link your `malloc()` and `free()` into some C program that you have or that you find on the net.  That is exactly the kind of real test we want but it presents challenges from a testing perspective. We want a test that doesn't require us to install another piece of software nor get its configuration correct.

I downloaded a nice piece of C software ([dparser](http://dparser.sourceforge.net/)) and instrumented the memory allocation by defining macros for `malloc()` and `free()` that made them instead call my routines. These routines printed a logging message and then delegated to the real C library routines. The output, if you look in `test/MALLOC_FREE_TRACE.trace` is a big file full of the following messages:

```
malloc 952 -> 0
malloc 8 -> 1
...
free 438
malloc 64 -> 438
free 439
...
```

It's a record of all of the memory allocation activity. In order to simulate a real program, all we have to do is build a little interpreter for this log/trace file. Upon `malloc n`, we call our `malloc(n)` function. The same for `free n`. When you run `test/test_replay.c`, it simulates the execution of that program from the perspective of memory allocation and deallocation.

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

You might need to install `valgrind` and `cmake` on your system. If you are using a Mac, do this:

```bash
$ brew update
$ brew install valgrind
$ brew install cmake
```

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

