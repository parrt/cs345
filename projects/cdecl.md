#Translating C declarations to English


## Goal

In this project, you will implement a small translator that recognizes a subset of C declarations and prints out the equivalent English meaning. For example, given input

```C
int *a[];
```

your translator must emit:

```
a is a array of pointer to int
```

You must fill in [Tool.java](https://github.com/USF-CS345-starterkits/parrt-cdecl/blob/master/src/cs345/cdecl/Tool.java) and the parse tree visitor[EnglishGenerator.java](https://github.com/USF-CS345-starterkits/parrt-cdecl/blob/master/src/cs345/cdecl/EnglishGenerator.java) in the starter kit.

Before you panic, note that my solution to the translator itself has 9 lines of Java code that I had to write myself. ANTLR and Intellij wrote the rest.  My solution for `Tool.java` has just 10 lines I had to write manually.

## Discussion

Please study [How To Read C Declarations](http://blog.parr.us/2014/12/29/how-to-read-c-declarations/) carefully. The rules for understanding C declarations are actually quite simple.

To implement this translator, we'll use ANTLR to parse a subset of C declaration syntax and then build a parse tree visitor that computes an appropriate English subphrase for each parse subtree. I have provided the grammar for you in [CDecl.g4](https://github.com/USF-CS345-starterkits/parrt-cdecl/blob/master/grammars/cs345/cdecl/CDecl.g4). The critical role is:

```
declarator
    :	declarator '[' ']'		# Array		// right operators have highest precedence
    |	declarator '(' ')'		# Func
    |	'*' declarator			# Pointer
    |	'(' declarator ')'		# Grouping
    |	ID						# Var
    ;
```

The parse trees resulting from this rule identify the various constructs and the order in which they should be applied. For example `*a[]` indicates we should apply the square bracket array operator first and then the pointer operator. We override that precedence with grouping parentheses if we want: `(*a)[]`.

I suggest using the ANTLR 4 plug-in for Intellij to get familiar with the grammar. Tell it to test starting with rule `declaration`.  The following parse tree corresponds to `*a[]`:

<img src=images/array-ptrs.png width=250>

and this one corresponds to `(*a)[]`:

<img src=images/ptr-array.png width=250>

Notice how, in the first tree, the `[]` binds tighter to the `a`. In other words, it is closest to it in the tree with respect to tree node depth. The deepest operations are done first, followed by the second deepest, up the tree to the root. That is why the second parse tree means "pointer to an array", because `*` is the deepest operator.

To implement your translator, you will create a visitor with methods for each of the `declarator` rule alternatives as well as a few others. The tree structure affects the order in which these methods are called. In other words, the parser will drive your translator according to what it finds in the input. 

Your individual visitor methods must augment and return the text computed for its child(ren). For example, in the visitor method that handles `*`, you just have to visit the `declarator` child and add string `pointer to` to its return value. See the ANTLR 4 reference for more on how the visitors work.

### Building and testing

The build assumes Java 8.

I suggest that you use Intellij, which knows how to deal with maven (`mvn`) builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows.

From the command line, you can build and store a jar with all of your software as follows on UNIX:

```bash
$ mvn install
```

That stores the jar in the following location:

```
~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar
```

The main program is executed as follows from the commandline:

```bash
$ java -cp ~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar:$CLASSPATH \
  cs345.cdecl.Tool "int i;"
i is a int
$ java -cp ~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar:$CLASSPATH \
  cs345.cdecl.Tool "int *a[];"
a is a array of pointer to int
```

Or, using maven as follows:

```bash
$ mvn test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running cs345.cdecl.TestCDecl
Tests run: 15, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.154 sec

Results :

Tests run: 15, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.914 s
[INFO] Finished at: 2016-01-05T09:12:26-08:00
[INFO] Final Memory: 23M/310M
[INFO] ------------------------------------------------------------------------
```

## Deliverables

You must fill in `Tool.java` and `EnglishGenerator.java` to get a working solution that passes all [15 unit tests](https://github.com/USF-CS345-starterkits/parrt-cdecl/blob/master/test/cs345/cdecl/TestCDecl.java).  You get 90% for passing all the tests and the other 10% after I sign off on your code after review. I will submit a github *pull request* if there are changes to be made, otherwise I will simply mark you down as 100%. Any required changes must be done prior to submission of your next project or the last day of class, whichever comes first.

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project. For example, a single large commit of the entire project at once could be considered circumstantial evidence for academic dishonesty.
