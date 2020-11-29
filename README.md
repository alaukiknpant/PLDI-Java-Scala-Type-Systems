# Null Pointer References: The Billion Dollar Mistake, this time at the type level
##### Alaukik N Pant

## 1. Backgorund and Motivation

Tony Hoare called his invention of Null references his billion dollar mistake.[[1]](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/) Before we understand why, let us define the null type.

**Null type:** Null is an object that has any reference type, i.e. ```Null``` is assigned to a value that is a member of every type. Null is generic and exhibits the behaviour of a variable that has no value.

Note that null is different from the `undefined` type, which is a value that has been declared but not yet assigned a value. While the undefined type has been around for a while, the null type is an invention of Tony Hoare. Let us first understnad the term ```references``` in null pointer references. The term object, in object-oriented programming languages, is another name for a reference. Each object, i.e. reference, has an address. Aditionally, each object or reference is a type of pointer which needs to be initialized when declared. Each reference must be initialized with a value that aligns with type of the variable that it refers to. For example, in the expression ```val a: String = b```, b must be an string and your compiler type checks if such is the case. Null pointers were invented by Hoare to by-pass the requirement for the afformentioned type-checking. To better understand `null`, consider the following expression:

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

Here, we are trying to compare tha variable ```ptr```, that is initialized as `null`, with the string `"HIII"`. However, this program throws a null-pointer exception because we are trying to find a string object in `ptr`, which is actually null. While this example was mostly straightforward, when your code base gets large, null references get harder to keep track of. Hence, `null` pointers cause problems at the level of writing programs by making it harder for you to detect bugs associated with initializing variables as `null`. However, in the paper [Java and Scala’s Type Systems are Unsound](https://ilyasergey.net/YSC3208/_static/papers/null.pdf), Nada Amin and Ross Tate argue that Null pointers "causes the same problem for the same reasons, but at the type level". This report examines thier arguments and provides additional examples of null pointers causing problems at the type-level. 

To start with, let us understand type systems and thier importance.

### Type Systems
A type system is a system of judgement and inference rules. A judgement is a claim and inference rules are used to derieve judgements from other judgements that are valid.[[1]](https://ilyasergey.net/YSC3208/_static/lectures/PLDI-Week-09-typing.pdf) 

For example,  ```3``` is an integer is a judgement. In the expression ```var a = 6 + 5```, the type of ```a``` is an integer is an inference because we can infer that we will get an integer when we add two integers. A programming language with judgements and inference rules are called typed languages. Java and Scala are examples of statically typed languages. A language is said to be statically typed if its type is known at compile time instead of runtime. 

### Sound Type Systems
A type system is sound if it provides the guarantee that it will do what it says. For example, if a function in Scala is supposed to return a List, it will return a list and not a set. Type checking refers to the idea of verifying that the constraints associated with types are enforced either *statically* at compile time or *dynamically* at run time. For compiled language, Static Type checking is important because it can statically rule out run time errors such as adding a string to an integer. For example, the expression ```"a" + 42```  is deemed incorrect at compile time because you cannot add a string to an integer. Type checking is also important because it provides information about types of intermediate operators to the compiler, provides extra information that can be used in compiler optimization and much more. As a programming language designer, you want to make sure that your type system obeys your specifications, i.e. your type system is sound.

### Java and Scala's type systems are Unsound
To prove that Java and Scala's type system are unsound, all we have to show is an example of a violation of a specification of the language. This report examines a particular example of a violation of the type system in Scala, and hence, shows that Scala's type system is unsound. Java has a similar problem and you can find more about it [here](https://ilyasergey.net/YSC3208/_static/papers/null.pdf).

### Path Dependent Types in Scala

To understand the unsoundness of Scala's type system, let us revisit Scala's has Path Dependent Types. To understand what this is, consider the following trait mentioned in the paper:

```scala
trait Rectangle {
	type Vertex;
	def getList(v : Vertex) : List[Vertex]
}
```

Note that the trait `Rectangle` here does not define the implementation of the type ```vertex```. This vertex can be a set of tuples in a 2D graph or a set of triples in a 3D graph or even a class that defines the storage of a list of neighbours. All we know is that the trait ```Rectangle``` indicates the type of its vertices and the type of its vertices can differ in different instances of a class that implement this trait. Hence, if the variable ```a``` is of type Rectangle, then we say that it has an associated **path dependent type** ```a.Vertex```. If the variable `b` is also of type Rectangle, we cannot be sure that ```a.Vertex``` is of the same type as ```b.Vertex```. For this reason, Scala allows path dependent types to be dynamically determined. Hence, ```a.Vertex```, a path-dependent type, is different from static types like ```String``` or parameterized types like ```T```. Path-dependent such as ```a.Vertex``` are important when encoding information into types tha can only be known at runtime.[[3
]](https://danielwestheide.com/blog/the-neophytes-guide-to-scala-part-13-path-dependent-types/)

## 2. Careful examination of the first Scala Program broken

The way in which this paper found out about the unsoundness of Scala's type system was by breaking its type system. Hence, we carefully examine the first program that broke Scala's type system.

Using Scala's path dependent type, let us study an example of the unsoundness of Scala's type system.

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

We argue that Scala's type system wrongly predicts the possible results of any computation. In particular, if we execute this program in the Java Virtual Machine, we get the following error: ```ClassCastException: java.lang.Integer cannot be cast to java.lang.String```. Instead of the type-checker catching the problem of casting an integer to a string at **compile time**, this peice of code reveals that Scala is relying on the JVM to catch this error at **run-time**.

 
To understand the reason behind this, we will make severeal judgements (represented as [J]) and inferences (represented as [I]).

	
First, consider the method `upcast(lb, t)`

A. **[J]** The parameters of upcast are the following:

   i. `lb`: a value whose type member is a **supertype of `T`** 
	
   ii. `t`: a value of type `T`
	
Notice that upcast upcasts `t` to the `lb`'s type member by using the supertype constraint. Next, conider the line with ```LowerBound[T] with UpperBound[U]``` in the code above. Here, we can infer the following judgement.

B. **[J]** The variable `bounded` is 

   i.  **a supertype of `T`** as it satisfies LowerBound[T]. - bounded satisfies the requirements of `lb` in judgement number **A.i**.

   ii. a subtype of U as it satisfies UpperBound[U].

Hence, we can pass the variables ```t``` and ```bounded``` unto upcast as done in the line ```return upcast(bounded, t)```. These judgements lead us to the following inference.

C. **[I]** Since the variable `bounded` is a super-type of `T` and a sub-type of `U`, `T` is a subtype of `U`. In other words, **`T` <: `bounded` <: `U` => `T` <: `U`**.

### Null Pointer Refernces Satisfies Inference Number C
From the inference rule, we learn that not only do we need bounded to be a a super-type of `T` and a sub-type of `U`, but also for `U` to be a super-type of `T`. Unfortunately, it is impossible to find a variable that satisfies this. However, Scala has implicit nulls that can be assigned to any reference type and that is exactly what is done in the code above, which leads us to the compilation of this code. Unfortunately, it leads to the creation of a **Non-Sense type**.

### Nonsense types and their problems

When we run this program,, we can pass an integer and a string because of the following sequence of logic that is problematic:

1. Consider ⊥ as the a subtype of everything
3. Consider ⊤ is a supertype of everything. 
4. There can exist a variable v that is ⊤ ≤ V ≤ ⊥
5. We know that Integer ≤ ⊤
6. We also know that ⊥ ≤ String 
7. Hence, Integer ≤ ⊤ ≤ V ≤ ⊥ ≤ String (This is problematic.)

The creation of V is referred to as a “non- sense” type and it leads to the ```CastCastException``` mentioned above.

### Potential Solution

The problem in the afformentioned example was caused by using null pointers in path dependent types. Hence, the authors propose that the solution would likely be to intorduce additional compile time analysis of path-dependent variables so that we have to do less checks during run-time. Other solutions discussed included either abandoning null pointers all together or abandoning them when using path-dependent variables. However, the authors recognize that this can be problematic in industry and argue that more research has to go into amending type-argument inference rules. Unfortunately, there seems to be no quick fix as the Scala team was able to identify other sources of unsoundness in Scala's type system that are related to this example but using other advanced features than null pointers.[[4]](https://ilyasergey.net/YSC3208/_static/lectures/PLDI-Week-09-typing.pdf)  


## 3. Evaluation of an Additional Example 

We have created an example of code that gives us a `ClassCastException` when you execute it on the JVM. This program is an additional example of why Scala's type system is unsound:

```scala
object unsound {
  trait Graph[T] {
    type vertex >: T
  }
  val g: Graph[Int] {type vertex <: List[Int]} = null
  def upcast(a: Graph[Int], x: Int): a.vertex = x
  def coerce(arg: Int): List[Int] = upcast(g, arg)

  def main(args: Array[String]): Unit = {
    val argument: List[Int] = List(1, 2, 3, 4, 5)
    val array: List[Int] = coerce(5)
  }
}
```

Lets us examine what is going on:

Here, we have created a trait ```Graph``` with a Path-Dependent-Type called `vertex`, where `vertex` is a super-type of a a generic type `T`. Then we create a variable `g` of type graph whose `vertex` is a sub-type of a List of Integers.

If `x` is the vertex of the graph, then `x` has to follow the following parameters:

1. **[J]**`x` is a super-type of Integers.
2. **[J]**`x` is a sub-type of an List of Integers.

From these two judgements, we can infer the following:

3. **[I]** `Int` <: `x` <: `List[Int]` => `Int` <: `List[Int]`

Obviously, `x`, here, is a non-sense type and as it is impossible to find a type `x` in Scala's specification such that `Int` <: `x` <: `List[Int]`. But assigning it as null helps us trick the type-checker.

Note that in the `upcast()` method above, we assign `a.vertex`, where `a` is of type graph to be an Integer. However, when we call `upcast(g, arg)` from the `coerce()` method on the graph `g` that expects its vertices to be sub-type a List of Integers, then we are inferring that `Int` <: `List[Int]`. In other words, we are trying to cast Integers to an immutable List. Hence, in runtime, JVM finds this to be problematic and returns the following message:

`Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to scala.collection.immutable.List`.


## 4. Conclusion

In this report, we have shown, using, atleast two examples that Scala's type system is unsound when using null pointer references with path dependent types. Java also has a similar problem when using generics with null pointer references. This bug took 12 years after the introduction of Java generics to uncover. It shows us that when different features of a programming language interact, i.e. null pointer references and path dependent types in this case, then we can have problematic scenarios while type-checking. Hence, programming language designers should not only think about the new features they design in isolation, but also the result of different features interacting.

