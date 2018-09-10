
# What is ClOS?

Common Lisp Object System port to R6RS Scheme.

CLOS, an acronym for Common Lisp Object System, is a standard set of extensions to the Common Lisp language to help people do object-oriented programming in Lisp. Scheme, as you know, is a dialect of Lisp with a simpler, more consistent syntax than Common Lisp's. 

The only dependency is [surfage](https://github.com/dharmatech/surfage).

# Port History

> 1992 Xerox Corporation  -Original Code

> 2007 Christian Sloma -Port to R6RS

> 2008? Mosh Scheme team -Packaged for Mosh

> 2010 dharmatech -Port to another Implantation

> 2018 Guenchi -Repackage and write document for Raven



# Introduction

CLOS vs. other approaches to OOP

The most popular object-oriented languages today (e.g. C++ and Java) share much of their syntax and much of their philosophy. CLOS have a Lisp-like syntax, very different from the block-structured syntax of the other languages mentioned above. They share the notions of run-time polymorphism (i.e. a function that works in several different ways depending on the kinds of objects to which it is applied), inheritance, etc. with essentially all other OO languages. And like most OO languages, CLOS consider every "object" to be an element of a "class", which may be written in terms of one or more "superclasses".
The most significant difference between CLOS and the other languages mentioned above is that in CLOS, a polymorphic function looks and behaves like an ordinary function, not tied to any one class of objects. By contrast, every polymorphic function in C++, Java, et al 'belongs' to one particular class, and must be invoked in conjunction with an instance of that class. For example, suppose there were a class named dial and a polymorphic function named turn, and we wished to turn the dial ThisDial to setting 200. A C++ programmer would write 

`ThisDial.turn (200); `

while a CLOS programmer would write 

`(turn ThisDial 200) `

The distinction is not merely syntactic: in C++, an action that involves multiple objects must "belong" to one of them, with the rest as parameters. For another example, suppose there are two classes named LightBulb and socket, and we wish to write a polymorphic function that (among other things) puts a light bulb into a socket. In C++ (or Java or ...), the programmer must choose whether to write a method for light bulbs, taking a socket as a parameter, or a method for sockets, taking a light bulb as a parameter: 

`ThisBulb.PutIn (ThatSocket); `

`ThatSocket.PutIn (ThisBulb); `

whereas the CLOS programmer simply writes 

`(PutIn ThisBulb ThatSocket)` 

In effect, a polymorphic function (called a "generic function" in CLOS terminology) is an ordinary function with multiple definitions, which automatically chooses the most appropriate definition at run time based on the classes of its arguments. No one argument is singled out as "the object" to which the method applies, and there is little need for the C++ construct called a "friend", a function applied to an object of one class which nonetheless has access to the private information of objects of another class.

This choice has several advantages, as pointed out above. It also has disadvantages: since a method belongs not to one specific class but to a combination of classes, it is much more difficult to control the visibility of methods, and the public/protected/private distinction in C++ cannot be applied to methods. Whether you consider these disadvantages to outweigh the advantages is a personal, almost religious, decision. For more discussion of this issue, see the OOP FAQ part 1, item 1.19.

Classes and Objects in CLOS

In CLOS, as in most object-oriented languages, every "object" is an element of one or more "classes", whose definitions may be derived from the definitions of other "superclasses". The behavior of an object is determined by its class(es): here "behavior" refers to
instance variables, i.e. information associated with each object in the class
methods, i.e. how various functions are implemented when applied to objects in the class
and possibly other things like class or pool variables which we sha'n't discuss at this point
By convention, class names in CLOS are surrounded in angle-brackets, e.g. <object>, <person>.

Creating instances

To create a new instance of an existing class, use the make function:

`(define sam (make <person>))`

creates a new instance of the <person> class and binds the Scheme variable sam to it. Some classes are defined in such a way (see the initialize function below) that additional arguments can be provided to the make function to determine properties of the new instance, e.g. to initialize its instance variables. For example, one might define the <person> class in such a manner that 


This manuel is modified from Tiny CLOS Tutorial

https://home.adelphi.edu/sbloch/class/archive/272/spring1997/tclos/tutorial.html
