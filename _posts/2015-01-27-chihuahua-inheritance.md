---
layout: post
title: "Chihuahua Inheritance"
date: 2015-01-27 14:45
---
So, in August of 2014, my roommate happened upon a chihuahua in the middle of nowhere on a Saturday night in the pouring rain. This creature was then gifted to me (quite literally &ldquo;Here, have a dog!&rdquo;). 

Now, I&rsquo;m not one for small dogs... especially chihuahuas. They&rsquo;re often neurotic and way too yippy (also, chihuahuas, as I was soon to find out, are pretty much guaranteed to have horrible breath). So I entered this relationship with this dog a skeptic. I wasn&rsquo;t even convinced she was a real dog at first, since she didn&rsquo;t eat or drink anything for about 24 hours - not even bacon. We surmised she must accept AAA batteries somewhere instead of food. 

As time wore on, however, it became very clear that this was not a typical small dog or even a typical chihuahua (other than the horrible breath). She (Tazza) barely makes a sound ever, except when she wants to come inside - and even then it is one short bark. She is affectionate and behaved, other than occasional humping of one particular red blanket. She likes running, eats normally, and when she really wants something, she stands on her hind legs and waves her front paws together like an Italian saying &ldquo;Bello, bello, belloooooo...&rdquo;. 

So naturally, when doing the [C# koans](https://github.com/CoryFoy/DotNetKoans) for Nashville Software School, I thought it would only be appropriate to make a special instance of the chihuahua class for Tazza. The C# koans begin teaching class inheritance via a base class of Dog, which looks like:

        1. public class Dog
        2. {
        3.     public string Name { get; set; }
        4.
        5.     public Dog(string name)
        6.     {
        7.         Name = name;
        8.     }
        9. 
        10.    public virtual string Bark()
        11.    {
        12.        return "WOOF";
        13.    }
        14.}

All well and good - dogs have a name and they bark. The subclass of Chihuahua is then inherited from the Dog base, overriding the string of &ldquo;Bark&rdquo; to return &ldquo;yip&rdquo; and adding the method Wag():

        1. public class Chihuahua : Dog
        2. {
        3.     public Chihuahua(string name) : base(name)
        4.     {
        5.     }
        6. 
        7.     public override string Bark()
        8.     {
        9.         return "yip";
        10.    }
        11.
        12.    public string Wag()
        13.    {
        14.        return "Happy";
        15.    }
        16.}
