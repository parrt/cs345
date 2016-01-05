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

You must fill in [Tool.java]() and the parse tree visitor[EnglishGenerator.java]() in the starter kit. 

## Discussion

First, read [How To Read C Declarations](http://blog.parr.us/2014/12/29/how-to-read-c-declarations/)  carefully.

### Building and testing


```bash
$ mvn install
```

`~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar`

The main program is executed as follows:

```bash
$ java -cp ~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar:$CLASSPATH \
  cs345.cdecl.Tool "int i;"
i is a int
$ java -cp ~/.m2/repository/edu/usfca/cs345/cdecl/1.0/cdecl-1.0.jar:$CLASSPATH \
  cs345.cdecl.Tool "int *a[];"
a is a array of pointer to int
```


## Deliverables

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).
 You are required to show git commits to your repository that are consistent with developing the software for this project. For example, a single large commit of the entire project at once could be considered circumstantial evidence for academic dishonesty.
