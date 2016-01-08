# Building a grammar and symbol table for a TypeScript subset

## Goal

Your goal is to build a grammar and symbol table for a subset of [TypeScript](http://www.typescriptlang.org/) (JavaScript with static types) The symbol table will have classes, functions, and variable definitions. Then you will make another pass over the parse tree and identify references to symbols.

## Discussion

There are three primary pieces: a grammar, a definition phase that walks the parse tree, and a reference phase. As an example, consider [class-inherit.ts](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/samples/class-inherit.ts):

```TypeScript
class Animal {
	name : string;
}

class Horse extends Animal {
}

class Dog extends Animal {
	breed : string ;
}
```

Your grammar should result in a parse tree that looks something like the following (truncated) tree.

<center>
<img src=images/typescript-classdef.png width=280>
</center>

Your definition phase will emit output according to [class-inherit.defs](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/samples/class-inherit.defs):

```
global
    Animal
        Animal.name
    Horse
    Dog
        Dog.breed
```

Your reference phase will emit output according to [class-inherit.refs](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/samples/class-inherit.refs):

```
5:20 class reference global.Animal
8:18 class reference global.Animal
```

**You will use ANTLR's [antlr/symtab](https://github.com/antlr/symtab) symbol table library** to make your lives easier.

### TypeScript grammar

You must create a grammar by examining the [examples provided for unit testing](https://github.com/USF-CS345-starterkits/parrt-symtab/tree/master/samples) and using the typescript grammar(s) you find around on the net, either for ANTLR or from the [language specification](http://www.typescriptlang.org/Content/TypeScript%20Language%20Specification.pdf). Your grammar must recognize all of the sample input but should not recognize more than those structures. In other words, your grammar should recognize a strict subset of TypeScript. You cannot just dump in a full TypeScript grammar you find on the net. It must be tight to match only the structures in the samples. My solution has 70 lines.

**You are specifically allowed to copy and paste from any existing grammar you find on the net as long as you attribute where you got it from. You may not copy from fellow students!**

This part of the project is deliberately ill-specified so that you can learn how to abstract a grammar from exemplars. Please note that the decisions you make about the grammar rule names and various structures, affect the parse tree listeners used in the next two phases.

So that we can annotate the parse tree with `Scope` pointers, we will add `returns` clauses to rules in the grammar that represent simple table scopes. You can see that I have done that for the global scope in [TypeScript.g4](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/grammars/cs345/typescript/TypeScript.g4):

```
program returns [Scope scope] : sourceElement* EOF ;
```

The parse tree node for `program` will have a `scope` field that you can set with `ctx.scope = ...;` from within a listener method where `ctx` is the listener method argument. Once you set this in the definitions phase, the references phase can set the current scope according to the value of that field.  For more information on this approach see the callout box in the ANTLR 4 reference manual called "Adding fields to nodes via rule arguments and return values."

Your grammar should support both the line and block comments.

### Defining symbols and scopes

As a reference, please see pattern 19: "Symbol table for classes" in the Language Implementation Patterns book.

Here are the operations to perform:

|Upon|Action(s)|
|----|---------|
program entry| create and push a global scope
program exit | pop scope
`class T` | create and define a class symbol, set ctx.scope, push scope
`class T extends U` | Same as `class T` but set superclass to `U`
exit class | pop scope
method entry | define method, set `ctx.scope`, push scope
method exit | pop scope
enter block | push local scope, set `ctx.scope`
 exit block | pop scope
def field x | define field symbol in current scope
def variable x | define variable symbol in current scope
def parameter x | define variable symbol in current scope

One of the key ideas is to maintain a `currentScope` field within the listener object.

### Referencing symbols

In the definition phase, you have created all of the necessary scopes, annotated the parse tree with them, and to find all of the symbols. Now, let's perform operations to maintain the `currentScope` field by accessing the scopes annotating the parse tree created in the previous pass:
 
|Upon|Action(s)|
|----|---------|
program entry| set current scope to `ctx.scope`
program exit | pop scope
class entry | set current scope to `ctx.scope`
exit class | pop scope
method entry | set current scope to `ctx.scope`
method exit | pop scope
enter block | set current scope to `ctx.scope`
 exit block | pop scope

In the same listener, we need to respond to references to identifiers:

|Upon|Action(s)|
|----|---------|
`class T extends U` | ref `U`
`x = ...;` | ref `x`
`... x ...` in expression | ref `x`
`f(...)` | ref `f`

## Getting started

I have provided a [starter kit](https://github.com/USF-CS345-starterkits/parrt-symtab) that you can pull into your repository.

You can get any updates to the starter kit by simply `git pull`ing again from `USF-CS345-starterkits/parrt-symtab.git`.

## Building and testing

The build assumes Java 8.

I suggest that you use Intellij, which knows how to deal with maven (`mvn`) builds. You can run the unit tests with a simple click. It should work on UNIX (including Mac) and Windows.

From the command line, you can build and store a jar with all of your software as follows on UNIX:

```bash
$ mvn install
```

To test the software from the command line, do the following.

```bash
$ mvn test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running cs345.typescript.TestDefs
Tests run: 18, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.179 sec
Running cs345.typescript.TestRefs
Tests run: 18, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.018 sec

Results :

Tests run: 36, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.639 s
[INFO] Finished at: 2016-01-08T11:43:04-08:00
[INFO] Final Memory: 22M/367M
[INFO] ------------------------------------------------------------------------
```

## Deliverables

You must complete the [TypeScript.g4](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/grammars/cs345/typescript/TypeScript.g4) grammar and [DefScopesAndSymbols](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/src/cs345/typescript/DefScopesAndSymbols.java), [RefSymbols](https://github.com/USF-CS345-starterkits/parrt-symtab/blob/master/src/cs345/typescript/RefSymbols.java) parse tree walks.

**You are specifically allowed to copy and paste from any existing grammar you find on the net as long as you attribute where you got it from. You may not copy from fellow students!**

## Submission

You must submit your project via github using your account and the repository I've created for you in organization [USF-CS345-S16](https://github.com/USF-CS345-S16).

You are required to show git commits to your repository that are consistent with developing the software for this project.

