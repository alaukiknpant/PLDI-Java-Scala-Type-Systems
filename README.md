# Null Pointer References: The Billion Dollar Mistake, this time at the type level
##### Alaukik N Pant

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

### Example of Scala's Unsoundness

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

We argue that Scala's type system wrongly predicts the possible results of any computation. In particular, if we execute this program in the Java Virtual Machine, we get the following error: ```ClassCastException: java.lang.Integer cannot be cast to java.lang.String```. Instead of the type-checker catching the problem of casting an integer to a string in **compile time**, this peice of code reveals that Scala, for now, is relying on the JVM to catch this error at **run-time**.

 
To understand the reason behind this, we will make severeal judgements (represented as [J]) and premises (represented as [P]).

	
First, consider the method `upcast(lb, t)`

A. **[J]** The parameters of upcast are the following:

   i. `lb`: a value whose type member is a **supertype of `T`** 
	
   ii. `t`: a value of type `T`
	
Notice that upcast upcasts `t` to the `lb`'s type member by using the supertype constraint. Next, conider the line with ```LowerBound[T] with UpperBound[U]``` in the code above. Here, we can infer the following judgement.

B. **[J]** The variable `bounded` is 

   i.  **a supertype of `T`** as it satisfies LowerBound[T]. - bounded satisfies the requirements of `lb` in judgement A.i

   ii. a subtype of U as it satisfies UpperBound[U].

Hence, we can pass the variables ```t``` and ```bounded``` unto upcast as done in the line ```return upcast(bounded, t)```. These judgements lead us to the following premise.

C. **[P]** Since the variable `bounded` is a super-type of `T` and a sub-type of `U`, `T` is a subtype of `U`. In other words, **`T` <: `bounded` <: `U` => `T` <: `U`**.

### Null Pointer Refernces Satisfies the Premise C
From the inference rule, we learn that not only do we need bounded to be a a super-type of `T` and a sub-type of `U`, but also for `U` to be a super-type of `T`. Unfortunately, it is very hard to find a variable that satisfies this. Unfortunately, Scala has implicit nulls that can be assigned to any reference type and that is exactly what is done in the code above, which leads us to the idea of the creation of a **Non-Sense type**.


### Nonsense types and thier problems


