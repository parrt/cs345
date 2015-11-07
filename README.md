# CS345 Programming Languages

The University of San Francisco

*WARNING: Under construction*

## Abstract

Because programming languages are at the core of how we communicate with machines, programmers should have a thorough understanding of how languages are designed, implemented, and manipulated. In this course, students learn to implement the key elements of programming languages such as virtual machine interpreters, type systems, symbol tables, memory allocation, garbage collection, and parsing.

Students also study various programming paradigms such as: object-oriented programming, functional programming, and declarative programming.  We will also give threaded programming a thorough treatment.

I expect students to be familiar with C, Java, and Python as we will and implement projects with those languages and also study some of the nooks and crannies of those languages.

## Requirements

CS245 is required background and is critical to the successful completion of the class. Undergraduates will have had Java and some Python (CS110) and C (CS220). Students must be comfortable with:

* Writing medium-sized programs in Java
* Small Python programs, classes, objects, functions
* Core C language syntax, pointers, arrays, structs, functions

## Administrivia

**ROOM.** Lo Schiavo Science 307.

**TIME.** TR 12:45pm - 2:30pm,  January 26 (Tue) - May 12 (Thu).

**EXAMS.** There will be 3 exams but no final exam.

### Instruction Format

Class periods of 1:45min each 2 times per week for 15 weeks. Instructor-student interaction during lecture is encouraged.

### Books

There is no textbook required for this class, but you will find The ANTLR 4 reference book useful. It's free [online](http://0-proquest.safaribooksonline.com.ignacio.usfca.edu/book/programming/9781941222621) via USF’s “safari online” as part of your tuition. Log in via the USF library site.

Unfortunately the more useful book, [Language Implementation Patterns](http://amzn.com/193435645X), is not available for free (not sure why). It’s not required for CS345, but I will closely follow some of that book in class and it’s cheap at $24 in printed form. (A reminder that using an unpaid-for copy of the electronic version is uncool and violates our academic honesty policy and is illegal.)

### Grading

| Artifact | Grade Weight | Due date |
|--------|--------|--------|
|[Java REPL](https://github.com/parrt/cs652/blob/master/projects/Java-REPL.md)| 5%| Feb 4 |
|[Java sampling and tracing profilers](https://github.com/parrt/cs345/blob/master/projects/profilers.md)| 10%| Feb 4 |
|[C declaration to English translator]() | 10% | |
|[malloc/free (C)]() | 10% | |
|[Virtual Machine (C)]() | 10% | |
|Quizzes | 5%| sporadic |
|Exam 1| 15%| Mon, March 30 |
|Exam 2| 15%| Wed, May 13 |
|Exam 2| 15%| |

*I expect to see proper git commit messages and github usage so I can track your development.*

I consider an "A" grade to be above and beyond what most students have achieved. A "B" grade is an average grade for a graduate student or what you could call "competence" in a business setting. A "C" grade means that you either did not or could not put forth the effort to achieve competence. An "F" grade implies you did very little work or had great difficulty with the class compared to other students.

Projects must run exactly as specified. Make sure that you do not have hardcoded files/directories in your code, remember that UNIX is case-sensitive as is Java, file names and class names must be correct, specified method signatures must be correct, etc...

Code quality counts. Even if you have perfect functionality, I will deduct points for poor and sloppy coding.

*I will be very strict and set a high standard in my grading*, but I will work hard to help you if you are having trouble. You some of you may not get the grade you were hoping for in this class, but I will do everything I can to make sure you learn a lot and have a satisfying educational experience!

Unless you are sick or have a family emergency, I will not change deadlines for projects nor exam times. For example, I will not give you a special final exam just because you want to fly home early. Consult the university academic calendar before making travel plans.

**ABOUT ME.** My name is Terence Parr and I’m a professor in the computer science department.  Please call me Terence or Professor (the use of “Terry” is a capital offense). For more information on me, see [http://parrt.cs.usfca.edu](http://parrt.cs.usfca.edu).

**TARDINESS.** Please be on time for class. It is a big distraction if you come in late.

**ACADEMIC HONESTY.** You must abide by the copyright laws of the United States and academic honesty policies of USF. You may not copy code from other current or previous students. All suspicious activity will be investigated and, if warranted, passed to the Dean of Sciences for action.  Copying answers or code from other students or sources during a quiz, exam, or for a project is a violation of the university’s honor code and will be treated as such. Plagiarism consists of copying material from any source and passing off that material as your own original work. Plagiarism is plagiarism: it does not matter if the source being copied is on the Internet, from a book or textbook, or from quizzes or problem sets written up by other students. Giving code or showing code to another student is also considered a violation.

The golden rule: **You must never represent another person’s work as your own.**

My policy is as follows:

 1. The first observed incident of cheating will result in a zero on the quiz or the assignment (for example). It will be reported to both the CS chair and the CS program assistant for tracking. 
 1. The second observed incident of cheating after the initial incident will result in a failing grade for the course.

If you ever have questions about what constitutes plagiarism, cheating, or academic dishonesty in my course, please feel free to ask me. I’m happy to discuss the issue in a forthright manner.

Official text from USF: *As a Jesuit institution committed to cura personalis—the care and education of the whole person—USF has an obligation to embody and foster the values of honesty and integrity. USF upholds the standards of honesty and integrity from all members of the academic community. All students are expected to know and adhere to the University's Honor Code. You can find the full text of the code [online](http://www.usfca.edu/catalog/policies/honor).*

**ON DISABILITIES.** If you are a student with a disability or disabling condition, or if you think you may have a disability, please contact USF Student Disability Services (SDS) at 415/422-2613 within the first week of class, or immediately upon onset of the disability, to speak with a disability specialist. If you are determined eligible for reasonable accommodations, please meet with your disability specialist so they can arrange to have your accommodation letter sent to me, and we will discuss your needs for this course. For more information, please visit http://www.usfca.edu/sds/ or call 415/422-2613.

## Syllabus

### Executing programs, evolution of program languages

### Programming paradigms

* [Functional programming and other fun stuff in Python](https://github.com/parrt/cs652/blob/master/lectures/functional-python.md)
* Object-oriented programming, [vtable impl](https://github.com/parrt/cs652/blob/master/projects/Java-vtable.md)
* Java generics
* SQL

### Parsing and grammars

### Symbol tables and static typing

### Memory management

* malloc/free
* automatic garbage collection

Bridging languages

### Virtual Machines

### Immutable/persistent data structures

### Threads