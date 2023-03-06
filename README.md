# Generic Boundaries

## Learning Goals

- Use generics boundaries in Java.
- Explain generic wildcards.

## The Problem

Sometimes when we create a method that accepts generics, we may still want to
restrict the types of objects that can be passed into the method because the
logic only makes sense for specific types of objects.

For example, we may have a generic method that needs to operate on numbers and
can operate on numbers of any type. Consider a method that adds all the numbers
in a list. Using unbounded generics, we might define this method as follows:

```java
public static <E> Double sumList(List<E> numberList) {
    double sum = 0.0;
    for (E element: numberList) {
        Number number = (Number)element;
        sum = sum + number.doubleValue();
    }

    return sum;
}
```

Note that in order to get the numerical value from our `numberList`, we need to
convert each element of the list to `Number`. This works fine if we call the
method with a list of objects of a type compatible with `Number`:

```java
public static void main(String[] args) {
    List<Number> numbers = new ArrayList<Number>();
    numbers.add(10.0);
    numbers.add(20);
    numbers.add(30);

    Double sum = sumList(stringNumbers);
    System.out.println("sum = " + sum);
}
```

But remember that one of the goals of using generics is to make our code more
flexible, while still remaining type safe. This means that we want the Java
compiler to be able to give us an error when we're not using the right types.

In this case, we can modify the code above to actually pass a `List` of `String`
objects instead of `Number` objects and the code will continue to compile fine:

```java
public static void main(String[] args) {
    List<String> stringNumbers = new ArrayList<String>();
    stringNumbers.add("0");
    stringNumbers.add("10");
    stringNumbers.add("20");

    // this code compiles
    Double sum = sumList(stringNumbers);
    System.out.println("sum = " + sum);
}
```

Running this code, however, gives us a `ClassCastException` because we cannot
cast a `String` into a `Number`:

```java
Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Number (java.lang.String and java.lang.Number are in module java.base of loader 'bootstrap')
```

The problem is with this line of code in the `sumList()` method:

```java
Number number = (Number)element;
```

## Using Generic Boundaries

This is where bounded generics can save the day! **Bounded generics** are type
parameters that can be restricted or bounded. We can restrict the generic types
that the method accepts by accepting a type and all of its subclasses (defining
an upper bound) or a type and all of its superclasses (defining a lower bound).

Let's see if we can define an upper bound for our `sumList()` method!

```java
public static <N extends Number> Double sumList(List<N> numberList) {
    double sum = 0.0;
    for (Number number : numberList) {
        sum = sum + number.doubleValue();
    }

    return sum;
}
```

Notice in the generic method above, we put the generic type definition,
`<N extends Number>`, right before the return type for the method. Also know
that in the case of generic bounds, when we use the word `extends`, it applies
to class extensions as well as interface implementations. So when we say
`<N extends Number`, we are saying that we expect the type to be a `Number` or
a class that extends `Number`.

This will now successfully compile as such:

```java
import java.util.List;
import java.util.ArrayList;

public class GenericBoundariesExample {

  public static <N extends Number> Double sumList(List<N> numberList) {
    double sum = 0.0;
    for (Number number : numberList) {
      sum = sum + number.doubleValue();
    }

    return sum;
  }

  public static void main(String[] args) {
    List<Number> nums = new ArrayList<Number>();
    nums.add(10);
    nums.add(20);
    nums.add(30);

    Double sum = sumList(nums);
    System.out.println("Sum is " + sum);
  }
}
```

Now if we were to change the `List` of numbers to be a list of `String` values,
we would get a compilation error prior to executing:

```text
GenericBoundariesExample.java:21: error: method sumList in class GenericBoundariesExample cannot be applied to given types;
        Double sum = sumList(nums);
                     ^
  required: List<N>
  found: List<String>
  reason: inference variable N has incompatible bounds
    equality constraints: String
    lower bounds: Number
  where N is a type-variable:
    N extends Number declared in method <N>sumList(List<N>)
1 error
```

### Wildcards

While the code example above with the upper bound does fix the issue at hand,
we typically would not write the method header that we did when using a
collection, like a `List`. Instead, we would make use of a wildcard. The
**wildcard**, represented by a question mark `?`, represents an unknown type.
Wildcards are pretty useful when working with generics and can be used as a
parameter type.

Here is how we can specify an upper bound wildcard for the generic type in our
`sumList()` method:

```java
public static Double sumList(List<? extends Number> numberList) {
    double sum = 0.0;
    for (Number number: numberList) {
        sum = sum + number.doubleValue();
    }

    return sum;
}
```

The syntax for specifying an upper bound wildcard is as follows:

- In the generic parameter type definition, we use the `? extends BaseType`
  notation where `?` is the wildcard and `BaseType` is the class that should be
  the upper bound of the parameter type.
    - In this case, the `BaseType` would be `Number` since that is the class we
      want to extend.

With the upper bound wildcard above, we are restricting the unknown type of the
`sumList()` method to classes that are subclasses of `Number`.

We can also specify a lower bound, which will restrict the unknown type of a
method to classes that are super classes of the class we specify.

Let's rewrite our `sumList()` method to only accept lists of `Integer` objects:

```java
public static Integer sumIntegerList(List<Integer> numberList) {
    int sum = 0;
    for (Integer number: numberList) {
        sum = sum + number;
    }

    return sum;
}
```

This would work to add a list of `Integer` objects:

```java
public static void main(String[] args) {
    List<Integer> integers = new ArrayList<Integer>();
    integers.add(10);
    integers.add(20);
    integers.add(30);

    Number sum = sumLIntegerist(integers);
    System.out.println("sum = " + sum);
}
```

But what if we wanted our `sumIntegerList()` method to be able to add integers
that are held in any class that is a superclass of the `Integer` class?

Let's first examine the `Integer` class hierarchy:

- `Integer extends Number` and
- `Number extends Object`, which yields the following hierarchy:
- `Object` -> `Number` -> `Integer`

Trying to call this `sumIntegerList()` method with a list of `Number` objects
like this:

```java
public static void main(String[] args) {
    List<Number> numbers = new ArrayList<Number>();
    numbers.add(0);
    numbers.add(10);
    numbers.add(20);

    Number sum = sumIntegerList(numbers);
    System.out.println("sum = " + sum);
}
```

Gives us the following error:

```plaintext
Example.java:21: error: incompatible types: List<Number> cannot be converted to List<Integer>
        Integer sum = sumList(nums);
```

This is because the type `List<Integer>` only matches the `Integer` class
specifically. For more flexibility, we can specify a lower bound for our generic
type like so:

```java
public static Integer sumIntegerList(List<? super Integer> numberList) {     
    Integer sum = Integer.valueOf(0);
    for (Object number: numberList) {
        sum += (Integer)number;
    }

    return sum;
}
```

The syntax for specifying a lower bound wildcard is as follows:

- In the generic parameter type definition, use the `? super Type` notation
  where `?` is the wildcard and `Type` is the class that should be the lower
  bound of the parameter type.

In this case, all objects that are passed into this method must be a `Integer`
or one of the classes it inherits from (directly, i.e. `Number` or indirectly,
which in this case means `Object`).

Also notice that we had to change the `for` loop slightly too to adjust for a
lower bound. If, in the event, we receive a Java `Object`, then we need to iterate
through the list of the Java `Objects`, and then explicitly cast it to an
`Integer`.

Now the following code compiles and runs:

```java
import java.util.List;
import java.util.ArrayList;

public class Example {

  public static Integer sumList(List<? super Integer> numberList) {
    int sum = 0;
    for (Object number : numberList) {
      sum = sum + (Integer)number;
    }

    return sum;
  }

  public static void main(String[] args) {
    List<Number> nums = new ArrayList<Number>();
    nums.add(10);
    nums.add(20);
    nums.add(30);

    Integer sum = sumList(nums);
    System.out.println("Sum is " + sum);
  }
}
```
