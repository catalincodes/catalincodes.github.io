---
layout: post
title: "Comparing Lists in C#"
date: 2025-12-23 20:15:00 -0400
categories: coding, development, dotnet
---

It's been a while that I have not written things here, and this is my first coding-related post on my blog. 

So what's it about? I had a classic issue, a few days ago and i thought I would share it with the rest of the world. In C#, we often compare properties, member variables, properties and member variables, you name it. What about lists though? It's tricky.
If I have two lists of integers and I try to ask if they are equal, the answer will be false:

```cs
List<int> listA = new list<int> { 1, 2, 3 };
List<int> listB = new list<int> { 1, 2, 3 };

var areTheyEqual = listA == listB; // False
```

In the above code we are actually comparing the location in memory of list A and list B, or as we say in the field, we are comparing references.

```cs
List<int> listA = new list<int> { 1, 2, 3 };
List<int> listB = listA;

var areTheyEqual = listA == listB; // True
```

So, how do we compare them? In the past, a long time ago, before .NET Framework 3.5 (2007), before LINQ, we would iterate over each entry and compare them. No one has to worry about this, because the vast majority of us that use C# have access to LINQ, so we can use [SequenceEqual](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sequenceequal) in order to compare the two lists:

```cs
List<int> listA = new list<int> { 1, 2, 3 };
List<int> listB = new list<int> { 1, 2, 3 };

var areTheyEqual = listA.SequenceEqual(listB); // True
```

This will work for lists of primitives like int , double, float, decimal and for strings. What about objects, though?

Suppose I have a potato object:

```cs
public class Potato
{
 public string Variety { get; set; }
 public double Weight { get; set; }
}
```

If I am to compare them using `SequenceEqual`, then, surprise:

```cs
List<Potato> listA = new List<Potato>()
{
    new Potato() { Variety = "Russet", Weight = 300.00 },
    new Potato() { Variety = "Idaho", Weight = 150.00 }
};

List<Potato> listB = new List<Potato>()
{
    new Potato() { Variety = "Russet", Weight = 300.00 },
    new Potato() { Variety = "Idaho", Weight = 150.00 }
};

var areTheyEqual = listA.SequenceEqual(listB); // False
```

It's still `False`. Why?

It's because SequenceEqual doesn't actually know how to compare the two objects, so it does what it does best, for object instances: it compares their memory address, their reference. 

Am I sure?

```cs
Potato russetPotato = new Potato() { Variety = "Russet", Weight = 300.00 };
Potato idahoPotato = new Potato() { Variety = "Idaho", Weight = 150.00 };

List<Potato> listA = new List<Potato>() 
{ 
 russetPotato, idahoPotato 
};

List<Potato> listB = new List<Potato>() 
{
 russetPotato, idahoPotato 
};

var areTheyEqual = listA.SequenceEqual(listB); // True
```

Great! So how do we compare them and make it work for the first example? Well, this is a special case, where as shown in the official [Microsoft Documentation for SequenceEqual](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sequenceequal), you need to teach the compiler how to compare potatoes.

```cs
public class Potato : IEquatable<Potato>
{
    public string Variety { get; set; }
    public double Weight { get; set; }

    public bool Equals(Potato other)
    {
        if (other == null) 
            return false;

        return this.Variety == other.Variety && this.Weight == other.Weight;
    }

    public override bool Equals(object obj) => base.Equals(obj as Potato);
    public override int GetHashCode() => (Variety, Weight).GetHashCode();
}
```

This is all that's required to make it work and to make it compare the two lists of custom-made objects.

![Selfie](/assets/it-aint-much-but-its-honest-work.jpg)

We are basically telling the compiler how to do a comparison between the objects, or simply put, when do we say return `True` for `idahoPotato1.Equals(idahoPotato2)`, and the answer is that if they have the exactly precise the same weight and same variety, we are looking at the same potato. In production, we would likely also have some kind of instance ID, but that's overkill for our example. 

Now, if you try to do `var areTheyEqual = listA.SequenceEqual(listB);` it will return True.

The last question is probably: what about the `GetHashCode()`. Great question. While it's not required for our case above, this is needed to make sure that if you ever need to use them in Hashtable, which will use `GetHashCode()` the two objects wil be considered equal based on their `Hash`. That's what I know, and I've not dealt too much with Hashtables so, take it with a grain of salt. If I ever do, you know that there will probably be another post. 

Until then, catch you on the next post, where we'll see what [catalin.codes](https://catalin.codes). 