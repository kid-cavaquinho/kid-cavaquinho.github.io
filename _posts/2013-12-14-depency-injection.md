---
layout:     post
title:      Dependency injection
date:       2013-12-14 14:21:19
summary:    "What is it and why should I care about it?"
categories: patterns
---

You might have heard of this term if you are into software development, a fellow called <a href="http://www.jamesshore.com/" target="_blank">James Shore</a> wrote something about it saying dependency injection (DI) "is a 25-dollar term for a 5-cent concept. That's not to say that it's a bad term..." and I agree with him. With some few words and simple pieces of code I will try to demonstrate the problems it tries to solve and the benefits that you can get from it. This concept is actually very simple and litelarally states an "injection" of a dependency from outside a class. 

### Wait, so what is a dependency? 

Let's imagine a musician, <a href="http://www.youtube.com/watch?v=LvmKlZGTTU4" target="_blank">Jimi Hendrix</a> came out of my mind now. To perform, your musician needs an instrument. That's a dependency. 

### Show me the code

If you are still listening to Jimi Hendrix, great. If not, no worries, bare with me for the next simple lines of code. The fellows of <a href="http://www.ninject.org/" target="_blank">Ninject</a>, an IoC container (we will get into IoC containers later on another post.) got this simple example written in C#:

<i>Let's say you're writing the next blockbuster game, where noble warriors do battle for great glory. First, we'll need a weapon suitable for arming our warriors.</i> (The weapon is the dependency on this example.)

{% highlight csharp lineanchors %}
class Sword
{
	public void Hit(string target)
	{
		Console.WriteLine("Chopped {0} clean in half", target);
	}
}
{% endhighlight %}

<i>Then, let's create a class to represent our warriors themselves. In order to attack its foes, the warrior will need an `Attack()` method. When this method is called, it should use its `Sword` to strike its opponent.</i>

{% highlight csharp lineanchors %}
class Samurai
{
	readonly Sword sword;
	public Samurai() 
 	{
		this.sword = new Sword();
	}

	public void Attack(string target)
    {
       	this.sword.Hit(target);
    }
}
{% endhighlight %}

<i>Now, we can create our Samurai and do battle!</i>

{% highlight csharp lineanchors %}
class Program
{
	public static void Main() 
    {
        var warrior = new Samurai();
        warrior.Attack("the evildoers");
    }
}
{% endhighlight %}
	
<i>As you might imagine, this will print Chopped the evildoers clean in half to the console. This works just fine, but what if we wanted to arm our `Samurai` with another weapon? Since the `Sword` is created inside the `Samurai` class's constructor, we have to modify the implementation of the class in order to make this change.

When a class is dependent on a concrete dependency, it is said to be tightly coupled to that class. In this example, the `Samurai` class is tightly coupled to the `Sword` class. When classes are tightly coupled, they cannot be interchanged without altering their implementation. In order to avoid tightly coupling classes, we can use interfaces to provide a level of indirection. Let's create an interface to represent a weapon in our game.

{% highlight csharp lineanchors %}
interface IWeapon
{
    void Hit(string target);
}
{% endhighlight %}
	
<i>Then, our `Sword` class can implement this interface:</i>

{% highlight csharp lineanchors %}
class Sword : IWeapon
{
    public void Hit(string target) 
    {
        Console.WriteLine("Chopped {0} clean in half", target);
    }
}
{% endhighlight %}
	
<i>And we can alter our `Samurai` class:</i>

{% highlight csharp lineanchors %}
class Samurai
{
    readonly IWeapon weapon;
    public Samurai() 
    {
        this.weapon = new Sword();
    }

	public void Attack(string target) 
    {
        this.weapon.Hit(target);
    }
}
{% endhighlight %}
	
<i>Now our `Samurai` can be armed with different weapons. But wait! The `Sword` is still created inside the constructor of `Samurai`. Since we still need to alter the implementation of `Samurai` in order to give our warrior another weapon, `Samurai` is still tightly coupled to `Sword`.
</i>
<i>Fortunately, there is an easy solution. Rather than creating the `Sword` from within the constructor of `Samurai`, we can expose it as a parameter of the constructor instead.</i>

{% highlight csharp lineanchors %}
class Samurai
{
    readonly IWeapon weapon;
    public Samurai(IWeapon weapon) 
    {
        this.weapon = weapon;
    }

    public void Attack(string target) 
    {
        this.weapon.Hit(target);
    }
 }
{% endhighlight %}

<i>Then, to arm our warrior, we can inject the `Sword` via the Samurai's constructor. This is an example of dependency injection (specifically, constructor injection). Let's create another weapon that our `Samurai` could use:</i>

{% highlight csharp lineanchors %}
class Shuriken : IWeapon
{
    public void Hit(string target)
    {
        Console.WriteLine("Pierced {0}'s armor", target);
    }
 }
{% endhighlight %}

<i>Now, we can create an army of warriors:</i>

{% highlight csharp lineanchors %}
class Program
{
    public static void Main() 
    {
        var warrior1 = new Samurai(new Shuriken());
        var warrior2 = new Samurai(new Sword());
        warrior1.Attack("the evildoers");
        warrior2.Attack("the evildoers");
    }
}
{% endhighlight %}

<i>This is called dependency injection by hand, because each time you want to create a `Samurai`, you must first create some implementation of `IWeapon` and then pass it to the constructor of `Samurai`. Now that we can change the weapon the `Samurai` uses without having to modify its implementation, the `Samurai` class could be in a separate assembly from `Sword`, in fact, we can create new weapons without needing the source code of the `Samurai` class!</i>

### Mighty conclusion

As you can see DI is really a simple concept and it will bring several advantages. First of all it allows the removal of hard-coded dependencies and changing them easily. This achievement represents loosely coupled classes. What this means is that your classes will have less knowledge about their dependencies. This makes our systems more flexible and maintainable. We also care about code quality and DI will help us with unit testing, for example. There are a few techniques to deal and apply it, but let's save that for other posts.

Note: Yes, I like Jimi Hendrix.
