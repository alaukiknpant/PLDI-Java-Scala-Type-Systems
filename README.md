# Null Pointer References: The Billion Dollar Mistake, this time at the type level
## Alaukik N Pant

Tony Hoare called his invention of Null references his billion dollar mistake.[[1]](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/) Before we understand why, let us define the null type.

**Null type:** Null is an object that has any reference type, i.e. ```Null``` is assigned to a value that is a member of every type. Null is generic and exhibits the behaviour of a variable that has no value.

Note that null is different from the `undefined` type, which is a value that has been declared but not yet assigned a value. While the undefined type has been around for a while, the null type is an invention of Tony Hoare. Let us first understnad the term ```references``` in null pointer references. The term object, in object-oriented programming languages, is another name for a reference. Each object, i.e. reference, has an address. Aditionally, each object or reference is a type of pointer which needs to be initialized when declared. Each reference must be initialized with a value that aligns type of the variable that it refers to. For example, in the expression ```val a: String = b```, b must be an string and your compiler type checks if such is the case. Null pointers were invented by Hoare to by-pass the requirement for the afformentioned type-checking. To better understand `null`, consider the following expression:

```scala
val a :String = null
```

Here, although ```null``` isn't a `String`, we were able to initialize `a` as `null`. We did not even have to type-check. As a result, Hoare's invention of the `null` type allowed your code to run quickly. Howeever, when he created the `null` type, he did not know that it compromised safety. In particular, your programs can crash at run-time throwing the famous `NullPointerException` when it tries to use a `null` instead of an object. The null type also enables memory leaks, security issues and viruses that could have led to loses ranging billions of dollars. To further solidify our understanding of the NullPointerExceptions, consider the following example:

```java
public static void main (String[] args) { 
    String ptr = null; 
    if (ptr.equals("HIII")) 
	System.out.print("Found HIII"); 
    else 
	System.out.print("Did not find HIII");        
} 
```

Here, we are trying to compare tha variable ```ptr```, that is initialized as `null`, with the string `HIII`. However, this program throws a null-pointer exception because we are trying to find a string object in `ptr`, which is actually null. While this example was mostly straightforward, when your code base gets large, null references get harder to keep track of. Hence, `null` pointers cause problems at the level of writing programs by making it harder for you to detect bugs associated with initializing variables as `null`. However, in the paper [Java and Scala’s Type Systems are Unsound](https://ilyasergey.net/YSC3208/_static/papers/null.pdf), Nada Amin and Ross Tate argue that Null pointers "causes the same problem for the same reasons, but at the type level". This report examines thier arguments and provides additional examples of null pointers causing problems at the type-level. 

To start with, let us understand type systems and thier importance.

#### Type Systems
A type is a system of judgement and inference rules. A judgement is a claim and Inference rules are used to derieve judgements from other judgements that are valid.[[1]](https://ilyasergey.net/YSC3208/_static/lectures/PLDI-Week-09-typing.pdf) 

For example,  ```3``` is an integer is a judgement. In the expression ```var a = 6 + 5```, the type of ```a``` is an integer is an inference because we can infer that we will get an integer when we add two integers. Java and Scala are both statically typed languages with a type system. A language is said to be statically typed if its type is known at compile time instead of runtime. A type system is sound if it provides the guarantee that it will do what it says. For example, if a function in Scala is supposed to return a List, it will return a list and not a set.

Type cehcking is important because it can statically rule out run time errors such as adding a string to a integer in the expression ```"a" + 42```.
Type checking also provides information about types of intermediate operators to the compiler, provides extra information that can be used in compiler optimization and much more.

The fact that static type checkers are conservative and can rule out possible run time errors in programming languages can also be a disadvantage. For example, your type checker may now allow a program that would eventually execute without an error. As a result, dynamically typed languages can be more expressive. Python and JavaScript are examples of dynamically typed languages. 

#### Path Dependent Types in Scala

Scala has Path Dependent Types. To understand what this is, consider the following trait:

```scala
trait Graph {
	type Vertex;
	def getNeighbors(v : Vertex) : List[Vertex]
}
```

The trait Graph here does not define the implementation of the type ```vertex```. This vertex can be a set of tuples in a 2D graph or a set of triples in a 3D graph or even a class that defines the storage of a list of neighbours. In this example, the type member of the trait ```Graph``` indicates the type of its vertices. If we are given a variable ```a``` of type Graph, it has an associated path dependent type ```a.Vertex```. Note that ```a.Vertex``` cannot be assigned to ```b.Vertex``` if ```b``` is another object og type Graph. Additionally, such type parameters cannot also be constrained to indicate that they can be a subtype/supertype of some type. 

Also, interistingly, Scala is allowing path dependent types to be dynamically determined. Unlike static types like ```String``` or parameterized types like ```T``` , ```a.Vertex``` is a path-dependent type. Path-dependent types are important when encoding information into types thacan be known at runtime.[[3
]](https://danielwestheide.com/blog/the-neophytes-guide-to-scala-part-13-path-dependent-types/)

#### Scala's Unsoundness

Using Scala's path dependent type, let us study the method ```coerce``` that can turn one type to another without (down)casting.

```scala
object unsound {
  trait LowerBound[T] {
    type M >: T; }
  trait UpperBound[U] { type M <: U;
  }
  def coerce[T,U](t : T) : U = {
    def upcast(lb : LowerBound[T], t : T) : lb.M = t
    val bounded : LowerBound[T] with UpperBound[U] = null
    return upcast(bounded, t)
  }
  def main(args : Array[String]) : Unit = {
    val zero : String = coerce[Integer,String](0) }
}

```

If we execute this program in the Java Virtual Machine, we get the following: ```ClassCastException: java.lang.Integer cannot be cast to java.lang.String```. 
To understand the reason behind this, consider the line with ```LowerBound[T] with UpperBound[U]``` in the code above.


#### Java Generics

Java introduced generics (parametric polymorphism) in 2005, that makes the life of a programmer easier in some sense. Generic methods are those method declarations that can be called on arguments of differnt types. For example, if you want to write a function extracts the head of a list, regardless of the types of the elements in the list, you would use a generic type. Note that the type parameter section in Java delimited by angle brackets `(<>)` to signify generics.

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



