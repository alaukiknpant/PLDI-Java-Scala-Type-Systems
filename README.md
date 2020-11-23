# A Reflection on Why [Java and Scala’s Type Systems are Unsound](https://ilyasergey.net/YSC3208/_static/papers/null.pdf)

### 1. Motivation and Background

A type is a system of judgement and inference rules. A judgement is a claim. For example, ```3``` is an integer is a judgement. Inference rules are used to derieve judgements from other judgements tht are valid.[1](https://ilyasergey.net/YSC3208/_static/lectures/PLDI-Week-09-typing.pdf) For example, consider the following peice of code:

```scala
var a = 6 + 5
```

Here, we can infer that the type of ```a``` is an integer because we can infer that we will get an integer when we add two integers. Java and Scala both have type systems that uses similar judgements as premises to come up with conclusions. Java and Scala are both statically typed languages. A language is said to be statically typed if its type is known at compile time instead of runtime. Python and JavaScript are examples of dynamically typed languages. 

A type system is sound if it does what its specifications says. For example, if a function in Scala is supposed to return a List, it will return a list and not a set.

A type system is sound if it actually succeeds at providing that guarantee. Thus informally a type system is sound if it ensures what its designers intended it to ensure


In this paper, we draw inspiration from the paper "Java and Scala’s Type Systems are Unsound" by Nada Amin and Ross Tate to shed light on this issue.




Type cehcking is important because it can statically rule out run time errors such as adding a string to a integer ```scala "a" + 42``` 


What is the problem that the paper solves?

Most type systems aim to provide some sort of guarantee about how a well-type program will behave

 Thus informally a type system is sound if it ensures what its designers intended it to ensure. This is much like programming: a program is correct if it does what it is supposed to do.
 
 Java’s type system is intended to ensure that if a method asks for an Integer, then it will only ever be given Integers, never Strings. 


- A function that can evaluate to or be applied to values of different types is known as a polymorphic function

- Scala and Java programs provide parametrically **polymorphic functions** that can turn any type into any type without (down)casting.

-  these programs demonstrate the unsoundness of Java and Scala’s current type systems.

-  Thus, in addition to valuing the development and verification of minimal calculi, our community should explore more ways to improve our chances of identifying abnormal interactions of features within reasonable time but without unreasonable resources and distractions.


Why is it important?


What were the existing approaches to address this problem (if any), an why they were found unsatisfactory by the authors?


2. Key Ideas and Contributions [3 points] Please, explain the following aspects of the paper.

What is the main methodology for solving the problem?
What are the challenges for implementing the solution?
Try to relate the ideas from the paper to the concepts explained in this class as you understand them. How are they extending what we have discussed in the lectures?

Points breakdown

##  use-site variance - Co-variant and Contravariant Use of a List of NUmbers

Suppose a method wants a List of Numbers. That method might 

1. get Numbers from the List - Covariant Use of a List of Numbers

  - When it is getting Numbers from the List, it could be provided a list of Integers or a List of Floats and the method works perfectly
  - This indicates that the method is a covariant use of the List of Numbers
  - it works given any List of any subtype of Number. 
  - type List<? extends Number>

2. add Numbers to the List

  -  method just giving Numbers to the List could just as easily be provided a List of Objects and the method would still work perfectly well.
  - this indicates that the method is a contravariant use of the List of Numbers 
  - it works given any List of any **supertype** of Number.
  - type List<? super Number>
  
 
## How does type-checking works

Consider the following Method Signature

```
<E> List<E> reverse(List<E> list) {...}
```

Consider the method ```reverse(ns)``` and the steps to typecheck this parameterically polymorphic methid

1. Find the type of ns in List<? extends Number>.
2. The type has a wildcard argument, so assign it a variable, say ```X```.
3. Change the type argument from the wildcard ```? extends Number``` that represents ```X```. So, our list has type ```List<x>```.



