
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


***Creating classes***		
		
 Creating a new class can be seen as simply creating a new instance of the predefined class `<class>`, e.g.		
		

```
 (define <person> (make <class>		
                        'direct-supers (list <object>)		
                        'direct-slots (list 'name 'age)))		
```

                        		
                        		
 creates a new class, initializing its "list of direct superclasses" to the one-element list (<object>) and its "list of direct slots" to (name age). However, creating a new class is such a common operation that they've provided a special make-class function to do the job. It takes two arguments: the list of direct superclasses (typically only one, until you start playing with multiple inheritance) and the list of names of "slots", or instance variables. Like all Scheme variables, these variables are untyped: you can equally well plug in the number 38, the symbol bluebird, the list (red green blue), or any other Scheme (or CLOS) object. 
	
With a syntax defined, the more simply way to create the <person> class above would be		
		
 `(define-class <person> (<object>) name age)`	

***Creating instances***

To create a new instance of an existing class, use the make function:

`(define sam (make <person> 'name "same" 'age 38))`

creates a new instance of the <person> class and binds the Scheme variable sam to it. Some classes are defined in such a way (see the initialize function below) that additional arguments can be provided to the make function to determine properties of the new instance, e.g. to initialize its instance variables. For example, one might define the <person> class in such a manner that 

	
		
 ***Instance Variables***		
		
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
Here we've defined a method which applies whenever the first argument is a <dial> and the second is a number. Its body is the lambda-form that takes two arguments named, dial and setting, and repeatedly turns up the dial until its position matches or exceeds the desired setting. (I assume that position-of and turn-up are already written somewhere.) 		 
   		
 You have no doubt noticed the unexplained "cnm" parameter to the lambda-form above. In most object-oriented languages, it is possible (and common) for a method to do almost the same thing as the corresponding method for a superclass, but with a little extra work before or after. To avoid having to rewrite all the code from the superclass's method, CLOS provides a way to invoke the superclass's method for the same generic. The Tiny CLOS implementation of this is, whenever a method is invoked, to give it a function named "call-next-method" as its first parameter, so that if it wishes to invoke the superclass's method for the same generic, it can do so by simply calling call-next-method. It is perhaps unfortunate that, even if you don't intend to use the call-next-method mechanism, every method must take an extra first parameter just in case. We'll discuss how to use this mechanism later.		
		
 In practice, you probably wouldn't bother defining this-method and then adding it to the generic turn; instead, you would probably do both in one step:		

```
 (add-method turn 		
         (make-method (list <dial> <number>)		
                      (lambda (cnm dial setting) 		
                           (while (< (position-of dial) setting)		
                                  (turn-up dial)))))		
```

 The initialize generic		
		
 Many object-oriented systems come with a variety of classes and methods already defined, and expect the user to create subclasses and override those methods as need be. One example is the initialize generic, which is called automatically whenever make creates a new instance of a class. The first argument to initialize is the object being created, and the second is a list of any extra arguments that were given to make.		
		
 I know that's a little confusing; here's an example that may help. Suppose we've defined the <person> class as above, and we want to provide a person's name (and, optionally, age) at the same time we create a <person> object. We might write		
		
 ```		
 (add-method initialize		
     (make-method (list <person>)		
         (lambda (cnm obj initargs)		
             (slot-set! obj 'name (car initargs))		
             (unless (null (cdr initargs))		
                     (slot-set! obj 'age (cadr initargs))))))		
 ```		
		
 Now we can create a person named "Sam" by typing		
		
 `(define friend1 (make <person> "Sam"))`		
		
 and another, 25-year-old, person named "Jeff" by typing		
		
 `(define friend2 (make <person> "Jeff" 25))`		
		
 Since a fairly common use of the initialize generic is simply to initialize named slots, I've written a subclass of <object> named <init-object> whose initialize method accepts a list of slot names and values; see initargs.scm for the (fairly simple) source code. A typical use would be		
		
 `(define <person> (make-class (list <init-object>) (list 'name 'age)))`		
 `(define pal (make <person> 'name "Jane" 'age 46))`		
		
 This violates the principle that users shouldn't know the names of slots; the <init-object> class is just a convenience for constructing illustrative examples quickly.		
		
 An example		
		
 Let's try a more complete example, one that doesn't require hypothetical functions to tell a robot arm to turn a dial. The problem at hand is to keep track of a collection of students, each of whom is registered for various courses at a hypothetical University in New York. The two most obvious objects in the problem description are "student" and "course", so let's create classes for them. For ease of initialization, let's just make them subclasses of <init-object>. In fact, since a <person> is already a subclass of <init-object> and already has a name and an age, let's make <student> a subclass of <person>, with only the additional information and methods that aren't already in the <person> class.		
   		
 ```  		
 (define <student> (make-class (list <person>)		
                   (list 'credits 'course-list)))		
 (define <course> (make-class (list <init-object>)		
                   (list 'name 'room 'time 'prof 'student-list)))		
 ```		
		
		
 Since we don't want users of these objects to know the slot names, we'll define some access functions:		
		
 ```		
 (add-method get-name		
      (make-method (list <student>)		
            (lambda (cnm student) (slot-ref student 'name))))		
 (add-method get-courses		
      (make-method (list <student>)		
            (lambda (cnm student) (slot-ref student 'course-list))))		
 (add-method get-class		
      (make-method (list <student>)		
            (lambda (cnm student)		
                (let ((credits (slot-ref student 'credits)))		
                     (cond ((< credits 30) 'freshman)		
                           ((< credits 60) 'sophomore)		
                           ((< credits 90) 'junior)		
                           (else 'senior))))))		
 (add-method get-name		
      (make-method (list <course>)		
            (lambda (cnm course) (slot-ref course 'name))))		
 (add-method get-room		
      (make-method (list <course>)		
            (lambda (cnm course) (slot-ref course 'room))))		
 (add-method get-time		
      (make-method (list <course>)		
            (lambda (cnm course) (slot-ref course 'time))))		
 (add-method get-prof-name		
      (make-method (list <course>)		
            (lambda (cnm course) (slot-ref course 'prof))))		
 (add-method get-roster		
      (make-method (list <course>)		
            (lambda (cnm course) (slot-ref course 'student-list))))		
 ```           		
                   		
 Notice that, although most of these access functions are simply calls to slot-ref, there's no reason they need be: if users are likely to want a student's class standing, yet the implementer decides to store the number of credits the student has completed, an "access function" like get-class may do significant work of its own.		
		
 Recall that we made <student> and <course> subclasses of <init-object> to allow easy initialization. However, let's insist that all students are created with no courses, and all courses are created with no students. We can do this by overriding their initialize methods:		
		
 ```		
 (add-method initialize		
     (make-method (list <student>)		
         (lambda (call-next-method student initargs)		
             (call-next-method)		
             (slot-set! student 'course-list ()))))		
 (add-method initialize		
     (make-method (list <course>)		
         (lambda (call-next-method course initargs)		
             (call-next-method)		
             (slot-set! course 'student-list ()))))		
 ``` 		
		
 In other words, even if some user does try to provide a course list or student list as an argument to make, they'll be set back to the empty list afterwards.		
		
 Of course, the main reason for keeping track of students and courses is for the former to take the latter. So we'd like to write two functions add and drop, each taking a student and a course: (add sam csc272) registers Sam for CSC 272, both adding Sam to the course roster for CSC 272 and adding CSC 272 to Sam's schedule, while (drop sam csc272) removes Sam from the course roster and CSC 272 from Sam's schedule.		
		
 Of course, things might go wrong. Sam might try to add a course for which he's already registered, or to drop a course for which he's not already registered, or to add a course that conflicts with another course he's already taking, etc. So we may need these functions to return some kind of error indication if they don't work. If they do work, let's have them return #f so the result can be tested easily, e.g. (if (add sam csc272) ...)		
		
 ```		
 (add-method add		
     (make-method (list <student> <course>)		
         (lambda (cnm student course)		
             (cond ((memv student (get-roster course))		
                    "Error: student already in course roster")		
                   ((memv course (get-courses student))		
                    "Error: course already in student's list of courses")		
                   (else (add-student student course)		
                         (add-course course student)		
                         #f)))))		
 (add-method drop		
     (make-method (list <student> <course>)		
         (lambda (cnm student course)		
             (cond ((not (memv student (get-roster course)))		
                    "Error: student not in course roster")		
                   ((not (memv course (get-courses student)))		
                    "Error: course not in student's list of courses")		
                   (else (remove-student student course)		
                         (remove-course course student)		
                         #f)))))		
 ```		
		
 These functions take care of the error-checking, but rely on four more (as yet unwritten) functions named add-student, add-course, remove-student, and remove-course to do the actual change. Since these four functions have to modify the internal state of <student> and <course> objects, we'll make them part of the interface to these classes:		
   		
 ```		
 (add-method add-student		
     (make-method (list <student> <course>)		
         (lambda (cnm student course)		
             (slot-set! course 'student-list		
                 (cons student (get-roster course))))))		
 (add-method add-course		
     (make-method (list <course> <student>)		
         (lambda (cnm course student)		
             (slot-set! student 'course-list		
                 (cons course (get-courses student))))))		
 (add-method remove-student		
     (make-method (list <student> <course>)		
         (lambda (cnm student course)		
             (slot-set! course 'student-list		
                 (delv student (get-roster course))))))		
 (add-method remove-course		
     (make-method (list <course> <student>)		
         (lambda (cnm course student)		
             (slot-set! student 'course-list		
                 (delv course (get-courses student))))))		
 ``` 

This manuel is modified from Tiny CLOS Tutorial

https://home.adelphi.edu/sbloch/class/archive/272/spring1997/tclos/tutorial.html
