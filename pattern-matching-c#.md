# Pattern Matching in C#

> Write smaller, more readable code and catch all boundary cases with pattern matching expressions
<div>
<style>
@scope {
    p {
        margin: 0;
    }
}
</style>
<div style="display: flex; justify-content: space-between; align-items: center;">
<time datetime="2024-03-26">March 26, 2023</time>
</div>
</div>

---

My original reason for writing this post was that, some time ago, I felt that the Microsoft documentation for pattern matching in C# did not fully communicate all the cool things you could do with pattern matching in everyday code. Since then, I have come across [this very succinctly summary of Pattern matching](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching) from Microsoft. However, I thought that this post could still serve benefit as an introduction in using pattern matching to make everyday code more concise.

Pattern matching, in short, is **a way you can test a value to see if it has certain characteristics**. In many languages, you can write the tests as *descriptions* of the object under test. Consider the below example:

```csharp
list is [0, 1, >3]
```

We haven't covered this specific syntax yet, but I can tell you this code resolves to either `true` or `false`. It could reasonably be inferred that the code checks the first two elements of `list` are `0` and `1`, and perhaps, that the third element is greater than `3`. Less noticably, it is also inferred that the `Count` of the `list` is `3`.

Patterns often let you extract data directly using the syntax of the pattern. Consider the below example.

```csharp
list is [0, 1, >3, int number, 4]
```

The above code (or *expression*) checks the first three elements like the prior example, but in addition to ensuring a `Count` of *five* elements, we now see an `int number` snuggled between the `>3` and the `4`.

The above code still resolves to `true` or `false`. The `list` may or may not match the pattern. However, if it does, our `int number` will now contain the element in between `>3` and `4`. This is what is meant above by using patterns to "extract data".

I should note here that the examples and styles presented in this article reflect my personal preference, and that I am not proposing an objective standard on a "better" way to write code.

# Nullable Reference Types
Firstly, to properly explain the benefit of pattern matching operators, I have to briefly mention Nullable Reference Types.

C# has had nullable value types (the struct ``Nullable<T>``) since C# 2.0, which serve as wrappers for value types which can hold ``null``. ``Nullable<T>``s are denoted by appending ``?`` to a value type, like so:

```csharp
public int Age { get; set; } // value type
public int? MaybeAge { get; set; } // nullable value type (Nullable<int>)
```

Nullable Reference Types (or NRTs, as they are called) are denoted by the same syntax of ``?``. But you may be confused as to why they exist, as reference types are innately nullable in C#. NRTs exist to **communicate that ``null`` is a valid value for a variable**. On the contrary, this communicates that ``null`` is **not** a valid value (or state, if you will) for variables of *non*-NRTs (so normal reference types). There have always been debates as to whether ``null`` should be accepted as a valid state for a variable, and null reference errors have been the source of countless bugs and hours of frustration in almost all programming languages since ``null``'s invention.

NRT-enabled projects make the validity of a "``null`` state" part of the variable's type. More than that, the developer actually gets some goodies from the compiler for giving it this extra information. With NRTs enabled, by default, the C# compiler will ensure that - barring ``System.Reflection``, Json deserialization, and cataclysmic world catastrophes - non-NRTs will be guaranteed to be ``null`` for the duration of the program.

```csharp
string name = null; // immediate warning, object is null by default, CS8600	Converting null literal or possible null value to non-nullable type.
string? maybeString = null;
```

As such, if the developer tries to "dereference" (use) a variable without having ensured it is not ``null``, [the compiler emits a warning](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/nullable-warnings).

```csharp
name = maybeString; // emits CS8600

if (maybeString != null)
    name = maybeString; // ✅ OK
```

It is worth noting here, that all null state checks associated with enabling NRTs are static and do not add runtime checks, which is why things like ``System.Reflection`` can invalidate the state.

The beauty of NRT is that nullable state can become part of your API definitions. Instead of peppering you code with repetitive null checks which can degrade performance and readability, you can define a point between your front-end and back-end precisely where `null`'s must be discarded, which means no more null checks and `ArgumentNullException.ThrowIfNull`s unless you have a strong reason to not trust the static analysis.

Fair warning: if this seems boring to you, the facets of pattern matching operators I will try to emphasize here might seem underwhelming. However, if this is a feature you'd put a ring on and honeymoon with, this next section should be where it "finally gets good".

# ``is`` and ``not``
You may have seen the following syntax before:
```csharp
var @object = GetSomeReferenceType();

if (@object is not null)
    ; // do something
```

This syntax obviously feels a lot like ``!=``, so the clever mind may start substituting ``null`` for other values on the right. They'd quickly find that this does not work for non-const variables, emitting a CS0150.

```csharp
record Person(string Name, int Age, Person? BestFriend);

Person kelly = new("Kelly", 40, null);
var person = GetSomePerson();

if (kelly is person) // CS0150: A constant value is expected
    ; // do something
```

I have received a similar error which essentially says the right-side of a pattern matching expression must be ``const`` *or an l-value*. While as of this writing, I have not been able to replicate this error message, I found that introducing the term l-value is informative as to how pattern matching expressions work.

An l-value stands for a "locator value" and generally refers to a variable declaration. That means anything (double-check me on this) that can be used to declare a variable can be used in a pattern matching expression.

```csharp
if (maybePerson is Person person)
    ; // do something

// or

if (maybePerson is var person)
    ; // do something
```

This syntax conveniently moves null-checking and assignment into a single expression.

```csharp
if (maybePerson != null)
   Person person = maybePerson;

// or

Person person;
if (maybePerson != null)
   person = maybePerson;

// vs

if (maybePerson is Person person)
   ; // ...
```

The null-state static analysis ensures that the scope of the variable matches what can be ensured by the logic of the code. For example, ``person`` is not valid outside of the ``if`` statement, since that is the only place it is guaranteed to not be ``null``.

```csharp
if (maybePerson is Person person)
   ; // ✅ use person here

// ❌ person is not valid outside of scope
```

What's really nice, is that that this pattern works with guard clauses too.

```csharp
if (maybePerson is not Person person)
   return;

// can use person here
```

This code might look a bit weird at first, since we expect a variable defined in an ``if`` statement (like an ``out`` parameter) to be valid within the scope of the corresponding block, but in the above example, ``Person person`` is actually being defined in the *outside* scope only. What we're really saying is "if this maybe-``null`` variable `maybePerson` is ``not null`` (it *does* conform to the non-NRT type), assign it to ``person`` and move on".

As David Fowler [pointed out](https://twitter.com/davidfowl/status/1606503770992832513), pattern matching (can) move assignment to the right.

You may have been thinking, "I use ``var`` a lot... can I use ``var`` with pattern matching?". And the answer is: you sure can! Though it may not be as useful as you think, since ``var`` corresponds to the *nullable* version of the inferred type.

```csharp
_ = person is var alsoPerson;

// translates to

_ = person is Person? alsoPerson;
```

The `null`-check comes from testing against the non-nullable type, which is only possible (as far as I am aware) when writing out the whole type name and not `var`. There is, however, a way ``var`` could be useful for property patterns, which are covered below.

# Multiple Inputs, List Patterns and ``when``

So far we have been using pattern matching to add some flair to our null checks - which I admit is the bulk of how I use pattern matching syntax these days. However, pattern matching itself is much more powerful and akin to [``match`` in F#](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/match-expressions) - which I will not get into as it would be a detour and I do not feel confident enough in F# to expound upon it.

The first way I have started using pattern matching in this section is with multiple inputs, something I like to call "bracket notation".

I like to describe pattern matching with `if` statements as defining the object you want to receive, then testing and assigning that object in one go according to your description, as opposed to testing a number of conditions on a variable and choosing to assign or discard it. Note the difference in the following case:

```csharp
Person person = new("Ken", 25, null);

if (person.Name == "Ken" && person.Age >= 18)
    ; // do something

// vs

if (person is { Name: "Ken", Age: >=18 })
    ; // do something
```

You can note that both of these examples both do essentially the same thing, but the pattern matching example is literally shorter and arguably more concise.

This bracket notation lets you describe what you want your input to conform to, instead of performing a series of Boolean checks. When you have a lot of properties and fields you want to check, this can - again, arguably - result in cleaner code.

Perhaps you are at the edge of your seat on this syntax. However, it is worth revisiting the non-`const` restriction above again, as the bulk of `if` checks performed are against non-`const` variables. Note the code below does not compile:

```csharp
int MinimumAge = 18; // non-const!

if (person is {Name: "Ken"} and {Age: >=MinimumAge}) // error CS0150: A constant value is expected
    ; // do something
```

Pattern matching expressions must conform to one of the patterns in [this C# patterns reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns).

**A variable is not a pattern**, so the correct version of the code above would be the following:

```csharp
int MinimumAge = 18;

if (person is {Name: "Ken"} and {Age: int age} && age >= MinimumAge)
    ; // do something
```

You may be thinking that this defeats the purpose of using a pattern matching expression in this instance, and you may be right.

On the other hand, you may find that making checks against several properties on an object with several layers (or nodes in a tree) of hierarchy may be convenient - particularly by using nested property patterns.

```csharp
if (person is { BestFriend: { Name: "Bob" } })
    ; // do something if person has a BestFriend named Bob
```

Take the below example, where we check if a `Node` on a tree has a certain structure:

```csharp
class Node
{
	public Node? Left { get; set; }
	public Node? Right { get; set; }
	public string Value { get; set; }
}

// ...

Node node = CreateSomeTree(); // could also be a Node of an existing tree

if (node.Left?.Left?.Right?.Value == "A"
	&& node.Left?.Left?.Value == "B"
	&& node.Right?.Value == "C"
	&& node.Left?.Value.Length > 0
	// you get the picture ...
	)
    ; // do something
```

In the above example, we reach down into a node to verify it has a certain structure, but the only way we can check that structure is with a series of checks, which are especially pesky and require us to add that `?` `null`-safe access operator to avoid `NullReferenceExceptions`. If we were to rewrite the above expression using pattern matching:

```csharp
if (node is { Left: { Left: { Right: { Value: "A" }, Value: "B" }, Value: { Length: >0 } }, Right: "C" }
	// ...
	)
    ; // do something
```

Because I tried to construct an example that shows the stark difference between the two syntaxes, I will not try to hide the fact that the latter expression is still quite hard on the eyes and may benefit from some line-breaks and tabs. However, by moving our series of assertions connected by `&&` into a single object description with sub-decriptions, we immediately cut down on visual redundancy from reaching into the object several times. If we still want to make Boolean checks on sub-properties use non-`const` variables, you can still declare any part of the expression as a variable and then add the manual check after the pattern matching expression.

```csharp
string nameWeCareAbout = GetNameWeCareAbout(); // non-const
if (node is { Left: { Left: { Right: { Value: "A" }, Value: "B" }, Value: { Length: >0 } } subNode, Right: "C" }
&& subNode.Value == nameWeCareAbout)
    ; // do something if node's Left node's Value is equal to nameWeCareAbout
```

## ``or``

Pattern matching includes the helpful ``or`` operator of almost self-explanatory use, however, you must remember that *pattern matching operators are not the same as Boolean operators*. Specifically, a pattern matching expression describes a single object, so ``or`` can only be used to describe alternate descriptions of the same object. Take the below example:

```csharp
if (person is {Name: "Ken"} or {Age: >=18})
    ; // do something
```

This makes certain types of operations much more concise. For example, since ``or``s can be stacked (as well as `and` and ``not``), checking whether a ``string`` is a certain value is as simple as ``name is "John" or "Kerry" or "Nami"``. However, it must be remembered that pattern matching operators are not 1-to-1 replacements for Boolean operators.

Referencing our earlier example, it should be noted that variable extraction is only possible with `or` or `not` insofar as it makes sense.

```csharp
if (person is {Name: "Ken"} or {BestFriend: Person bestFriend}) // error CS8780: A variable may not be declared within a 'not' or 'or' pattern.
    ; // do something with bestFriend
```

The above code will not compile because the `bestFriend` variable cannot be guaranteed to exist inside or outside the `if` statement, given that if `person` contains `Name: "Ken"`, the first part of the pattern (before the `or`) will satisfy the whole pattern and whether `bestFriend` is `not null` will never be checked, meaning the variable cannot be defined.

Property patterns are fun to play around with, as they can be used with the [null-conditional access operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) and can be stacked.

```csharp
if (root.Child?.Child.Child is { PropertyA: not null, PropertyB: >20})
   ; // do something
```

## List patterns

As the [C# patterns reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns) is comprehensive, I specifically would like to defer to [this section](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#list-patterns) to demonstrate slice and subpatterns with lists.

However, for want of an introductory example, I'd like to include how I used list patterns to parse simple string text [here](https://github.com/johnW-ret/project-3-dotnet/blob/master/Program.cs).

Program.cs
```csharp
string input = @"A	0	3
B	2	6
C	4	4
// ...
";

StringReader sr = new(input);
// ...

while (sr.ReadLine() is string line)
{
    string[] tokens = line.Split('	');

    if (tokens is [string name, string arrivalTimeToken, string durationToken]
        && int.TryParse(tokens[1], out int arrivalTime)
        && int.TryParse(tokens[2], out int duration))
    {
        Jobs.Add(new(tokens[0], arrivalTime, duration));
    }
}
```

In the above example, you can see how I
1. checked the length of the `string[]` is 3
2. assigned each element to a variable using a single pattern matching expression.

# `switch` Expressions, Positional patterns, and `_`

It may be further clearer to the reader by now that this blog post is not a holistic overview of pattern matching in C# - rather, a fairly involved introduction. I would encourage the reader to further consult the references linked above for exploring the cool and useful ways you can construct pattern matching expressions. Up until now, we have been using patterns inside `if` statements to perform checks and extract data, and while I find this is my most common use case for patterns, it's arguably the least powerful.

## `switch` Expressions

Consider the below code that I just had Bing Chat generate for me:
```csharp
// Define an enum for weather conditions
public enum Weather
{
    Sunny,
    Cloudy,
    Rainy,
    Snowy
}

// Define an enum for temperature ranges
public enum Temperature
{
    Hot,
    Warm,
    Mild,
    Cool,
    Cold
}

// Define a tuple for outerwear items
public struct Outerwear
{
    public string Hat;
    public string Jacket;
    public string Gloves;
}

// Write a switch statement that takes a weather and temperature value and assigns an outerwear value
public static Outerwear ChooseOuterwear(Weather weather, Temperature temperature)
{
    // Declare an outerwear variable to store the result
    Outerwear outerwear;

    // Use a switch statement to evaluate the weather and temperature combination
    switch (weather, temperature)
    {
        // For each possible case, assign an appropriate outerwear value
        case (Weather.Sunny, Temperature.Hot):
            outerwear = new Outerwear { Hat = "Sun hat", Jacket = "None", Gloves = "None" };
            break;
        case (Weather.Rainy, Temperature.Cool):
            outerwear = new Outerwear { Hat = "Rain hat", Jacket = "Raincoat and sweater", Gloves = "Waterproof gloves" };
            break;
        case (Weather.Snowy, Temperature.Cold):
            outerwear = new Outerwear { Hat = "Beanie and scarf", Jacket = "Winter coat and sweater", Gloves = "Woolen gloves" };
            break;
        case (Weather.Cloudy, _): // Use a discard pattern to match any temperature with cloudy weather
            outerwear = new Outerwear { Hat = "None", Jacket = "Warm jacket", Gloves = "None" };
            break;
        case (_, Temperature.Mild): // Use a discard pattern to match any weather with mild temperature
            outerwear = new Outerwear { Hat = "None", Jacket = "Light jacket", Gloves = "None" };
            break;
        // For any other case, throw an exception
        default:
            throw new ArgumentException("Invalid weather or temperature value");
    }

    // Return the outerwear value
    return outerwear;
}
```

You may find online many cases of people arguing about `switch` statements and `if else` / `if else if` statements. Some say that `if` statements easily introduce edge case bugs where certain uncovered cases fall through the cracks or into the wrong "bucket". `switch` statements - on the other hand - are not as versatile as Boolean conditionals in `if` statements, but their simplicity helps force the programmer to keep control flow simple. Additionally, a handy `default` case can be added to catch any uncovered cases.

Some complain that `switch` statements are unnecessarily restrictive and verbose and that edge case bugs can easily be caught using `if` statement guard clauses to filter out bad states one by one, instead of handling all possible states in a `switch` statement or an `if else` / `else if` set of blocks.

`switch` expressions are a possible solution to this dilemma in two parts:
1. They are concise and easy to read.
2. Uncovered cases produce compiler warnings

I asked Bing Chat to modify the above code to use `switch` expressions instead, and you may note the `ChooseOuterwear` method appears about half as tall.

```csharp
public static Outerwear ChooseOuterwear(Weather weather, Temperature temperature)
{
    // Use a switch expression to evaluate the weather and temperature combination
    return (weather, temperature) switch
    {
        // For each possible case, assign an appropriate outerwear value
        (Weather.Sunny, Temperature.Hot) => new Outerwear { Hat = "Sun hat", Jacket = "None", Gloves = "None" },
        (Weather.Rainy, Temperature.Cool) => new Outerwear { Hat = "Rain hat", Jacket = "Raincoat and sweater", Gloves = "Waterproof gloves" },
        (Weather.Snowy, Temperature.Cold) => new Outerwear { Hat = "Beanie and scarf", Jacket = "Winter coat and sweater", Gloves = "Woolen gloves" },
        (Weather.Cloudy, _) => new Outerwear { Hat = "None", Jacket = "Warm jacket", Gloves = "None" }, // Use a discard pattern to match any temperature with cloudy weather
        (_, Temperature.Mild) => new Outerwear { Hat = "None", Jacket = "Light jacket", Gloves = "None" }, // Use a discard pattern to match any weather with mild temperature
        // For any other case, throw an exception
        _ => throw new ArgumentException("Invalid weather or temperature value")
    };
}
```

Instead of switching on a value to execute a `case`, a `switch` expression works by resolving to a value associated with the first matching expression.

For example, `(Weather.Cloudy, _)` could resolve to
- `(Weather.Cloudy, Temperature.Hot)`
- `(Weather.Cloudy, Temperature.Cold)`
- any other `Temperature`

If the `_` expression were not included at the bottom, the compiler would automatically identify that not all cases are covered (`(Weather.Sunny, Temperature.Cold)`, for example) and produce a CS8509 warning.

```csharp
public static Outerwear ChooseOuterwear(Weather weather, Temperature temperature)
{
    // Use a switch expression to evaluate the weather and temperature combination
    return (weather, temperature) switch // The switch expression does not handle all possible values of its input type (it is not exhaustive). For example, the pattern '(Weather.Sunny, Temperature.Warm)' is not covered.
    {
        // For each possible case, assign an appropriate outerwear value
        (Weather.Sunny, Temperature.Hot) => new Outerwear { Hat = "Sun hat", Jacket = "None", Gloves = "None" },
        (Weather.Rainy, Temperature.Cool) => new Outerwear { Hat = "Rain hat", Jacket = "Raincoat and sweater", Gloves = "Waterproof gloves" },
        // ... but not _ and not all cases
    };
}
```

This behavior is similar to `default` in `switch` statements, except instead of just falling out of the `case`s block when no `case`s match and depending on the programmer to write some "catch-all" code, the `switch` expression **emits a warning** when you fail to cover all edge cases, and throws a runtime error when hitting the unsupported case if you ignore this warning.

One consequence of this is that those cases where your `switch` statements do nothing but set some other variable have become a lot simpler. A `switch` _expression_ resolves to a value, where a `switch` _statement_ conditionally executes some code. In the same way that `ChooseOuterwear` directly returns a value from the `switch` expression, you can also set a variable directly from the result of a `switch` expression.

```csharp
enum Access { Restricted, Moderated, Unrestrained }
// ...
var access = (age, isModerator) switch
{
	(<18, false) => Access.Restricted,
	(>=18, false) => Access.Moderated,
	(_, true) => Access.Unrestrained
};
```

## `when`
You may be wondering how to introduce custom Boolean checks as we did above with `if` statements. This is accomplished with the `when` keyword in `switch` expressions. Instead of trying to get Bing to shoehorn a `when` into our mutilated and transmogrified example, I encourage the reader to refer to [this example](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#var-pattern) in the C# reference which compares tuple inputs to each other.
```csharp
public record Point(int X, int Y);

static Point Transform(Point point) => point switch
{
    var (x, y) when x < y => new Point(-x, y),
    var (x, y) when x > y => new Point(x, -y),
    var (x, y) => new Point(x, y),
};
```

Furthermore, the `when` keyword can even be used in a try/catch block to conditionally catch exceptions!
[`when` in a `catch` statement - modified from Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when)
```csharp
try
{
    return await new HttpClient.GetStringAsync("https://localHost:10000");
}
catch (HttpRequestException e) when (e.Message.Contains("301"))
{
    return "Site Moved";
}
// ...
catch (HttpRequestException e)
{
    return e.Message;
}
```

What's fun about `switch` expressions, is that they allow you to let your data define parts of decision-making in your code. I'm not sure if I would necessarily call them "declarative" or "functional" (I'll let the F# devs argue about that), but the static analysis certainly cuts down on some of the uncertainty when using `if` statements.

The API that powers this very blog [uses `switch` expressions](https://github.com/johnW-ret/Retruate.Blog/blob/fce5fd868b926c8bc176b4bd51516b7d9fdc4dcc/Retruate.Blog/Pages/BlogPostPage.razor)! They work well with minimal APIs, which already support lambdas and expressions.

```csharp
group.MapGet("/", async (string key, IPostsTableAccess tableAccess, IPostClient postClient)
    => await tableAccess.GetRow(key) switch
    {
        Row row => await postClient.GetPost(row.Key) switch
        {
            { IsPublished: bool p } post when isEditor || p => Results.Ok(post),
            _ => Results.Problem(),
        },
        _ => Results.NotFound()
    });
```

You can use `switch` expressions [with razor code too](https://github.com/johnW-ret/Retruate.Blog/blob/master/Retruate.Blog/Pages/BlogPostPage.razor)!
```csharp
<div>
    @((RenderFragment)(State switch
    {
        FetchState.Success when Post is not null => @<BlogPost Post="Post" />,
        FetchState.Loading => @<div>Loading</div>,
        _ when Error is not null => @<div class="alert alert-danger">@Error</div>,
        _ => @<div class="alert alert-danger">Please try again.</div>
    }))
</div>
```

In this post, we didn't (necessarily) cover [positional patterns](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#positional-pattern), [recursive patterns](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/patterns), or using pattern matching for [type conversion](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns) such as with interfaces and inheritance. I would encourage the interested reader to check out these examples provided by Microsoft for building [type-driven and data-driven algorithms](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/pattern-matching).
