# Recognizing regular expressions

## Goal

Regular expressions are a very valuable tool because they allow programmers to quickly parse, filter, and extract data from text files.  To really understand regular expressions, it's a good idea to implement a simplified version, which is the goal of this project.  This project will also exercise the recursion muscle in your brain and teach you how to build a simple parser.

You will build a regular expression parser that converts the following regular expressions to NFA (Nondeterministic Finite Automata) and then implement a matching engine that simulates those NFA.

|regex|meaning|
|-----|-------|
a | match letter a
xy | match regex x then regex y
x \| y| match regex x or y
(x) | group regex
x* | zero or more of regex x
 | empty string matches nothing; e.g., (x\|) optionally matches x

White space is not allowed and only `a-z` (lowercase) letters are allowed as atoms. There is no escape character to get the meta-symbols (parentheses, star, and pipe).

## Discussion

The formal regular expression meta-language you must match is described by the following ANTLR grammar:

```
regex    : sequence ('|' sequence)* ;
sequence : closure sequence | ; // same as closure* but easier to build NFA
closure  : element '*' | element ;
element  : letter | '(' regex ')' ;
```

Yes, it's a grammar (ANTLR notation) that describes another grammar syntax (regex notation) that describes input languages.

### Parsing regular expressions

You will build a recursive-descent parser as methods in class `RegexParser`.

### NFA Construction

See "Converting Regular Expressions to NFAs" in [Russ Cox's amazing documents on regex](https://swtch.com/~rsc/regexp/regexp1.html). Here is the set of NFA constructions you must perform. These NFA were generated automatically by `Regex.toDOT()`

|regex|NFA|
|-----|-------|
 | <img src=images/epsilon.png width=80>
a | <img src=images/a.png width=80>
xy | <img src=images/xy.png width= 200>
x \| y|<img src=images/x_or_y.png width= 200>
(x \| y) | <img src=images/x_or_y.png width= 200>
x* | <img src=images/closure.png width=200>
(a\|b\|c)*|<img src=images/a_or_b_or_c.png width=320>
(a\|) | <img src=images/x_or_eps.png width=200>

## Getting started

I have provided a [starter kit](https://github.com/USF-CS345-starterkits/parrt-regex) that you can pull into your repository. From the command line, clone your project repo and then pull in my starter kit:

```bash
$ git clone git@github.com:USF-CS345-S16/USERID-regex.git
Cloning into 'USERID-regex'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
$ cd USERID-regex
$ git checkout -b master
Switched to a new branch 'master'
$ git remote add starterkit git@github.com:USF-CS345-starterkits/parrt-malloc.git
$ git pull starterkit master
remote: Counting objects: 28, done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 28 (delta 4), reused 25 (delta 4), pack-reused 0
Unpacking objects: 100% (28/28), done.
From github.com:USF-CS345-starterkits/parrt-regex
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> starterkit/master
$ git push origin master
Counting objects: 28, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (22/22), done.
Writing objects: 100% (28/28), 8.24 KiB | 0 bytes/s, done.
Total 28 (delta 4), reused 0 (delta 0)
To git@github.com:USF-CS345-S16/USERID-regex.git
 * [new branch]      master -> master
```

## Building and testing

The build assumes Java 8.

I suggest that you use Intellij, which knows how to deal with maven (`mvn`) builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows.

From the command line, you can build, test, and store a jar with all of your software as follows on UNIX:

```bash
$ mvn install
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running cs345.regex.TestDOTOutput
Tests run: 12, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.096 sec
Running cs345.regex.TestMatching
Tests run: 18, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.019 sec
Running cs345.regex.TestNFAConstruction
Tests run: 13, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.015 sec
Running cs345.regex.TestSyntaxErrors
Tests run: 6, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.004 sec
...
[INFO] Installing /Users/parrt/courses/CS345/projects/regex/target/regex-1.0.jar to /Users/parrt/.m2/repository/edu/usfca/cs345/regex/1.0/regex-1.0.jar
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.306 s
[INFO] Finished at: 2016-01-06T10:53:37-08:00
[INFO] Final Memory: 11M/310M
[INFO] ------------------------------------------------------------------------
```

Every time you commit to your repository, your software will automatically be downloaded and tested on the Travis continuous integration server using maven. Here is what a successful result looks like:

<img src=images/travis-regex.png width=500>

Check out [https://travis-ci.com/USF-CS345-S16/USERID-regex](https://travis-ci.com/USF-CS345-S16/USERID-regex) where USERID is your github user id. Mine is parrt, for example. You will not be able to see the repositories of other students.

## Deliverables

* All parser methods in [RegexParser.java](https://github.com/USF-CS345-starterkits/parrt-regex/blob/master/src/cs345/regex/RegexParser.java)
* Method [Regex.simulate](https://github.com/USF-CS345-starterkits/parrt-regex/blob/master/src/cs345/regex/Regex.java#L58)
* Method [toDOT\_](https://github.com/USF-CS345-starterkits/parrt-regex/blob/master/src/cs345/regex/NFA.java#L76)
* Method [NFA.getStates](https://github.com/USF-CS345-starterkits/parrt-regex/blob/master/src/cs345/regex/NFA.java#L96)

You must pass all tests in `tests` dir and show green on travis.

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

