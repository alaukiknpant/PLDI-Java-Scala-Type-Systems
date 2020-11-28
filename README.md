# A Reflection on Why [Java and Scala’s Type Systems are Unsound](https://ilyasergey.net/YSC3208/_static/papers/null.pdf)

#### Type Systems
A type is a system of judgement and inference rules. A judgement is a claim and Inference rules are used to derieve judgements from other judgements that are valid.[[1]](https://ilyasergey.net/YSC3208/_static/lectures/PLDI-Week-09-typing.pdf) 

For example,  ```3``` is an integer is a judgement. In the expression ```var a = 6 + 5```, the type of ```a``` is an integer is an inference because we can infer that we will get an integer when we add two integers. Java and Scala are both statically typed languages with a type system. A language is said to be statically typed if its type is known at compile time instead of runtime. A type system is sound if it provides the guarantee that it will do what it says. For example, if a function in Scala is supposed to return a List, it will return a list and not a set.

Type cehcking is important because it can statically rule out run time errors such as adding a string to a integer in the expression ```"a" + 42```.
Type checking also provides information about types of intermediate operators to the compiler, provides extra information that can be used in compiler optimization and much more.

The fact that static type checkers are conservative and can rule out possible run time errors in programming languages can also be a disadvantage. For example, your type checker may now allow a program that would eventually execute without an error. As a result, dynamically typed languages can be more expressive. Python and JavaScript are examples of dynamically typed languages. 

#### Java Generics

Java introduced generics (parametric polymorphism) in 2005, that makes the life of a programmer easier in some sense. Generic methods are those method declarations that can be called on arguments of differnt types. For example, if you want to write a function extracts the head of a list, regardless of the types of the elements in the list, you would use a generic type. Note that the type parameter section in Java delimited by angle brackets `(<>)` to signify generics.

Consider the following piece of Java code that the author presents that seems seemingly bad[[2]](https://hackernoon.com/java-is-unsound-28c84cb2b3f):

```java
List<Integer> ints = Arrays.asList(1);
List raw = ints;
List<String> strs = raw;
String one = strs.get(0);
```
Here, the variable ```one``` seems to allow a string although it is getting an integer. Althogh this behavious looks bad, it was intentional and the designers planned it such that it does runtime checks to verify the variable ```one``` is actually a string and throws a ClassCastException if not. In this way, by introducing generics, the designers of Java planned the type system of Java to be safe. But Nada Amin and Ross Tate argue in their paper that "Java and Scala’s Type Systems are Unsound".

Consider the following peice of code from the paper:

```java
class Unsound {
    static class Constrain<A, B extends A> {}
    static class Bind<A> {
	// T <: Z
	//     Z <: U
	// 	Explicit constraint on wildcard Implicit constraint on wildcard
	<B extends A>
	    A upcast(Constrain<A,B> constrain, B b) {
	    return b; }
    }
    static <T,U> U coerce(T t) {
	Constrain<U,? super T> constrain = null; Bind<U> bind = new Bind<U>();
	return bind.upcast(constrain, t);
    }
    public static void main(String[] args) {
	String zero = Unsound.<Integer,String>coerce(0); }
}
```
The Java type checker does not complain about this peice of code and this proram should compile without exception. However, when we run this program, we get the following error that compains that we have assigned an integer to a string variable. T


```
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
	at Unsound.main(Unsound.java:16)
```

### What happened in the Unsound Class?

The runtime checks mentioned in the first example above is only done on generic types and the Unsound class has no generic types.

Recall that there are no generic types in this peice of code



What is the problem that the paper solves?




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



