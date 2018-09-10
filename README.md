
# What is CLOS?

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

***CLOS vs. other approaches to OOP***

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

***Classes and Objects in CLOS***

In CLOS, as in most object-oriented languages, every "object" is an element of one or more "classes", whose definitions may be derived from the definitions of other "superclasses". The behavior of an object is determined by its class(es): here "behavior" refers to

instance variables, i.e. information associated with each object in the class

methods, i.e. how various functions are implemented when applied to objects in the class

and possibly other things like class or pool variables which we sha'n't discuss at this point

By convention, class names in CLOS are surrounded in angle-brackets, e.g. `<object>, <person>`.

***Creating instances***

To create a new instance of an existing class, use the make function:

`(define sam (make <person>))`

creates a new instance of the <person> class and binds the Scheme variable sam to it. Some classes are defined in such a way (see the initialize function below) that additional arguments can be provided to the make function to determine properties of the new instance, e.g. to initialize its instance variables. For example, one might define the <person> class in such a manner that 

Creating classes		
		
 Creating a new class can be seen as simply creating a new instance of the predefined class <class>, e.g.		
		
 ```		
 (define <person> (make <class>		
                        'direct-supers (list <object>)		
                        'direct-slots (list 'name 'age)))		
 ```                       		
                        		
                        		
 creates a new class, initializing its "list of direct superclasses" to the one-element list (<object>) and its "list of direct slots" to (name age). However, creating a new class is such a common operation that they've provided a special make-class function to do the job. It takes two arguments: the list of direct superclasses (typically only one, until you start playing with multiple inheritance) and the list of names of "slots", or instance variables. Like all Scheme variables, these variables are untyped: you can equally well plug in the number 38, the symbol bluebird, the list (red green blue), or any other Scheme (or Tiny CLOS) object. So the more common way to create the <person> class above would be		
		
 `(define <person> (make-class (list <object>) (list 'name 'age)))`		
		
 Instance Variables		
		
 In OOP in general, an "instance variable" is a variable associated with each individual instance of a class. In the above example, name and age are instance variables of the class <person> because each person has its own (possibly distinct) name and age. Instance variables may be viewed as a bigger, better version of the fields in a Pascal record or a C struct. The CLOS word for instance variable is "slot".		
 To get the value of a specified slot in a specified object, use the slot-ref function, which takes two parameters -- the object in question, and the name of the slot -- and returns the value of the slot:		
		
 `(slot-ref sam 'age) `		
 38		
		
 To change the value of a specified slot in a specified object, use the slot-set! function. Notice the exclamation point at the end of the function name, indicating that (unlike most Scheme functions) this one actually changes its arguments. It takes three parameters: an object, a slot name, and the new value. So if it were Sam's birthday, we might write		
		
 `(slot-set! sam 'age (+ 1 (slot-ref sam 'age)))`		
		
 It is generally considered good programming practice to treat slot names as implementation, rather than interface, so users of your object class don't access slots in your objects directly. This is for two reasons: first, if users start relying on your objects to contain slots with specific names, you lose the freedom to change the implementation by renaming or even eliminating some of those instance variables; and second, an object may contain several pieces of related information that must be kept consistent, an essentially impossible task if users can change one piece of information at a time behind your back. Accordingly, most CLOS classes are provided with "access functions" whose purpose is simply to give the user certain information about the object, without the user ever knowing how that information is stored (the slot name, or even whether it is stored in a slot at all). Another kind of access function allows the user to change certain information about an object, again without knowing how that information is stored. Which brings us to...		
		
 Generic functions and methods		
		
 Polymorphism is provided in CLOS by something called a generic function: a function with (potentially) several different definitions (methods), one of which is chosen at run-time based on the classes of the arguments.		
		
 Creating generic functions		
		
 You can create a new generic function in CLOS with the make-generic function, which takes no arguments:		
		
 `(define turn (make-generic))`		
		
 You'll probably use the function add-method, which adds a method to an existing generic, much more often. Indeed, if you apply add-method to a generic that hasn't already been defined with make-generic, it'll define it for you, so you actually never need make-generic at all.		
		
 `(add-method turn this-method)`		
		
 Creating and attaching methods		
		
 OK, so you can use make-generic or add-method to create a generic function, and (in the latter case) attach a new "method" to it. But what is a "method"? In a nutshell, a method is an ordinary Scheme function definition, together with information indicating what classes which arguments have to belong to in order for this method to be applicable. Methods are constructed by a function named make-method, which takes two arguments, a list of classes and a function (typically presented as a lambda-form). For example,		
		
 ```		
 (define this-method		
         (make-method (list <dial> <number>)		
                      (lambda (cnm dial setting) 		
                           (while (< (position-of dial) setting)		
                                  (turn-up dial)))))		
 ```
 

This manuel is modified from Tiny CLOS Tutorial

https://home.adelphi.edu/sbloch/class/archive/272/spring1997/tclos/tutorial.html
