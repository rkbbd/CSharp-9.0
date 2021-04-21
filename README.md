# CSharp-9.0
C# 9.0 is taking shape, and I’d like to share my thinking on some of the major features they are adding to this next version of the language.


# Examples of the majority of the new features in C# 9
1. [Top level statements](#1. Top-level-statements)
2. [Init only setters](#2. Init-only-setters)
3. [Record types](#3. Record-types)
4. [Pattern matching enhancements](#4. Pattern-matching-enhancements)
5. [Target-typed new expressions](#5. Target-typed-new-expressions)
6. [With-expressions](#6. With-expressions)
7. [Static anonymous functions](#7. Static-anonymous-functions)
8. [Attributes on local functions](#8. Attributes-on-local-functions)
 
 Reference https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9
 https://gist.github.com/dannyskoog/c9a61620a6f00430d8b728ef481164cc#file-8-attributes-on-local-functions-cs
 https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/



## 1. Top level statements

Top-level statements remove unnecessary ceremony from many applications. Consider the canonical "Hello World!" program:

```
// C# 8
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            System.Console.WriteLine("Hello World!");
        }
    }
}

// C# 9
System.Console.WriteLine("Hello World!");
```
If you wanted a one-line program, you could remove the using directive and use the fully qualified type name


## 2. Init only setters
you can create init accessors instead of set accessors for properties and indexers. Callers can use property initializer syntax to set these values in creation expressions, but those properties are readonly once construction has completed

```
var person1 = new Person("Danny", "Skoog"); // OK
var person2 = new Person("Danny", "Skoog") { FirstName = "Mike" }; // OK
person1.FirstName // NOT OK

public class Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }

    public Person(string firstName, string lastName) => (FirstName, LastName) = (firstName, lastName);
}

```

## 3. Record types
Init-only properties are great if you want to make individual properties immutable. If you want the whole object to be immutable and behave like a value, then you should consider declaring it as a record.

```
// Declaration
public record Person(string FirstName, string LastName);

// Equality
var person1 = new Person("Danny", "Skoog");
var person2 = new Person("Danny", "Skoog");
Console.WriteLine(person1 == person2); // Outputs "True"

// Immutability
var person3 = new Person("Danny", "Skoog"); // OK
var person4 = new Person("Danny", "Skoog") { FirstName = "Mike", LastName = "Johnson" }; // OK
var person5 = person1 with { FirstName = "Mike" }; // OK
person3.FirstName = "Mike"; // NOT OK

// ToString
Console.WriteLine(person1.ToString()); // Outputs "Person { FirstName = Danny, LastName = Skoog }"

// Deconstruction
var (firstName, lastName) = person1;

```
## 4. Pattern matching enhancements

C# 9 includes new pattern matching improvements:
  1. Type patterns match a variable is a type
  2. Parenthesized patterns enforce or emphasize the precedence of pattern combinations
  3. Conjunctive and patterns require both patterns to match
  4. Disjunctive or patterns require either pattern to match
  5. Negated not patterns require that a pattern doesn't match
  6. Relational patterns require the input be less than, greater than, less than or equal, or greater than or equal to a given constant.

```
// Relational patterns
// C# 8
bool IsLetter(char c) => c >= 'a' && c <= 'z' || c >= 'A' && c <= 'Z';
// C# 9
bool IsLetter2(char c) => c is >= 'a' and <= 'z' or >= 'A' and <= 'Z';



// Logical patterns - Example 1
// C# 8
static int GetSpeed3(Vehicle vehicle)
{
    if (!(vehicle is null))
    {
        if (!(vehicle is Bus))
            return 100;

        return 70;
    }

    return 0;
}
// C# 9
static int GetSpeed4(Vehicle vehicle)
{
    if (vehicle is not null)
    {
        if (vehicle is not Bus)
            return 100;

        return 70;
    }

    return 0;
}

// Logical patterns - Example 2
// C# 8
static int GetSpeed5(Vehicle vehicle) => !(vehicle is Bus) ? 100 : 70;
// C# 9
static int GetSpeed6(Vehicle vehicle) => vehicle is not Bus ? 100 : 70;

// Parenthesized patterns
// C# 9
bool IsLetter3(char c) => c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z');

```

## 5. Target-typed new expressions

Do not require type specification for constructors when the type is known.

```
// Example 1
// C# 8
Person person = new Person();
// C# 9
Person person2 = new();

// Example 2
// C# 8
DoIt(new Person());
// C# 9
DoIt(new());

// Example 3
// C# 8
Person GetPerson()
{
    return new Person();
}
// C# 9
Person GetPerson2()
{
    return new();
}

```
## 6. With-expressions

When working with immutable data, a common pattern is to create new values from existing ones to represent a new state. For instance, if our person were to change their last name we would represent it as a new object that’s a copy of the old one, except with a different last name. This technique is often referred to as non-destructive mutation. Instead of representing the person over time, the record represents the person’s state at a given time.

```
var otherPerson = person with { LastName = "Hanselman" };

```
With-expressions use object initializer syntax to state what’s different in the new object from the old object. You can specify multiple properties.
A record implicitly defines a protected “copy constructor” – a constructor that takes an existing record object and copies it field by field to the new one:
```

protected Person(Person original) { /* copy all the fields */ } // generated

```

## 7. Static anonymous functions

```
var x = 5;

// C# 8
Func<int, int> func = (int num) => x + 5; // OK
// C# 9
Func<int, int> func2 = static (int num) => x + 5; // NOT OK

```

## 8. Attributes on local functions
Attributes with a specified meaning when applied to a method, its parameters, or its type parameters will have the same meaning when applied to a local function, its parameters, or its type parameters, respectively.
A local function can be made conditional in the same sense as a conditional method by decorating it with a [ConditionalAttribute].
```
int Calculate(int? num)
{
    int CalculateLocal([DisallowNull] int? num)
    {
        return num.GetValueOrDefault() + 10;
    }

    return CalculateLocal(num);
}

Calculate(100);

```
