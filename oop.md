##Encapsulation
[encapsulation](http://www.sapphiresteel.com/Blog/What-Is-Encapsulation-and-does-it.html)
**Some points:**

This was the code in controversy:
```ruby
I pointed out that encapsulation is broken when a programmer assigns a variable (let’s call it x) and passes it to a method (e.g. someOb.someMethod( x )) inside which the value of x is modified and the programmer then uses this modified value - e.g.

x = 10
someOb.someMethod( x ) # <= let&#8217;s suppose this method changes x to 20
y = x * 2 #<= Now y is 40!
```

My assertion that this broke encapsulation turned out to be somewhat controversial. For example, in a thread on reddit, one writer commented: “Modifying objects passed into a method by calling methods on them does not break encapsulation, it’s the heart of what OO is about, creating side effects.”

#### Some quotes about encapsulation:

> ”No component in a complex system should depend on the internal details of any other component.” That simple, succinct sentence tells you everything you need to know about encapsulation. What this implies is that it should be possible substantially to rewrite the implementation of a method without having any effects (what I would call ‘side-effects’) on any code that ‘calls’ that method.

>“A message must be sent to an object to find out anything about it... This is needed because we don’t want the form of an object’s inside known outside of it.”

> In other words, the key, the central idea of what we now call ‘encapsulation’ is not merely data-hiding, but implementation-hiding. You don’t need simply to hide information (variables) from the world beyond the object - you also want to hide behaviour (methods). If the implementation details of a method have any effect of any sort on code outside of that object, then encapsulation is broken.

>“Encapsulation is a great bonus from the point of view of the user of an object - they do not need to know anything about the object’s implementation, only what its published protocols are.”
(Smalltalk and Object Orientation: An Introduction - John Hunt)
### Ways to break encapsulation

Here are a few examples of ways in which you can easily break encapsulation (that is, you can make external code dependent on the internal implementation details of objects ) in Ruby:

1. Modifying the value of an argument inside a method breaks encapsulation
```ruby
class C
 def aMethod( aVar )
   aVar << "hello"
   return aVar.reverse
 end
end

ob1 = C.new
mystring = ["world"]
```


2. Global Variables Break Encapsulation

```ruby
$x = 100

class C
 def aMethod
   $x = 200
 end
end

class C2
 def anotherMethod
   return $x * 2
 end
end

ob1 = C.new
ob2 = C2.new
```

In the above, ob2 and ob1 are objects of different classes. Calling methods of one object should have no effect when calling methods of the other. But they do, thanks to the reference to the global variable, $x. In fact, the results of my code change according to the order in which the methods are called...

```ruby
puts ob1.aMethod   #<= 200
puts ob2.anotherMethod   #<= 400

puts ob2.anotherMethod   #<= 200
puts ob1.aMethod   #<=200
```

That, in effect, ‘exposes’ the implementation details of each class’s methods. If I change the code inside them, the effects will ripple through to unrelated objects!
