---
layout: post
title:  "Composition > Inheritance"
date:   2014-07-06 00:00:00
---

When an aspiring developer starts learning OOP, a lot of courses focus extensively on inheritance.
However, many design patterns avoid inheritance and use composition instead.
This is because inheritance can get you in trouble when changes need to be made, since it violates the principle of encapsulation. 
Or in other words the implementation of the base and derived classes are tightly coupled.
Meaning a change in one class can affect other classes.
Lets explore an example of inheritance design, see how it can get us in trouble down the road when changes need to be applied and how to solve this problem using composition.

<!--more-->

### Inheritance

Suppose we create an application which let certain animals wag their tails. The initial application would look like this:

Class Diagram:

![](/assets/posts/2014/07/Inheritance.jpg)

Code:

{% highlight java linenos %}
public class Animal {
     public void wagTail() {
          System.out.println("Wag wag wag"); 
     }
}

public class Dog extends Animal {
}

public class Cat extends Animal {
}

public class Test {
     public static void main(String[] args) {
          Animal dog = new Dog();
          Animal cat = new Cat();
                    
          dog.wagTail(); //Result: Wag wag wag
          cat.wagTail(); //Result: Wag wag wag
     }
}
{% endhighlight %}


### Requested changes
Everything is fine so far, but now we have to implement a couple of changes:

1. Support for gorillas
2. Have the cat's tail wagging act differently from the dog
     
A gorilla doesn't have a tail, but it is an animal. We could still inherit from animal and override the wagTail method with an empty method, but we would have to do this for every animal we'll add which doesn't have a tail. At some point we could have multiple animals without tails containing the empty overridden wagTail method. When we have to add functionality for the tailless animals we would have to duplicate this for every tailless animal class. 

As for change 2, we could again override the wagTail method for the Cat class and put the new implementation in the Cat class itself. In this case we would have to duplicate this code when we have to add other catlike animals.

### Composition

A core design principle of OOP is separating aspects that vary from what stays the same. In this case the tail wagging implementation needs to be separated from the base class. We can accomplish this by using composition. The animal class will delegate the actual tail wagging implementation to a separate concrete class via an interface. 

Class diagram:

![](/assets/posts/2014/07/Composition.jpg)

Code:

{% highlight java linenos %}
public  interface ITailWagging {
     public void wagTail();
}

public class HappyTailWagging implements ITailWagging {
     public void wagTail() {
          System.out.println("Wag wag wag. I'm happy.");
     }
}

public class AnnoyedTailWagging implements ITailWagging {
     public void wagTail() {
          System.out.println("Wag wag wag. I'm annoyed with you.");
     }
}

public class NoTailWagging implements ITailWagging {
     public void wagTail() {
          // do nothing  
     }
}

public class Animal {
     private ITailWagging tailWagging;
     
     public void wagTail() {
          //delegate tail wagging to ITailWagging class
          tailWagging.wagTail();
     }
     
     public void setTailWagging(ITailWagging tailWagging) {
          //change tail wagging implementation at runtime
          this.tailWagging = tailWagging;    
     }    
}

public class Dog extends Animal {
     public Dog() {
          tailWagging = new HappyTailWagging();
     }
}

public class Cat extends Animal {
     public Cat() {
          tailWagging = new AnnoyedTailWagging();
     }
}

public class Gorilla extends Animal {
     public Gorilla() {
          tailWagging = new NoTailWagging();
     }
}

public class Test {
     public static void main(String[] args) {
          Animal dog = new Dog();
          Animal cat = new Cat();
          Animal gorilla = new Gorilla();
          
          dog.wagTail(); //Result: Wag wag wag. I'm happy.
          cat.wagTail(); //Result: Wag wag wag. I'm annoyed with you.
          gorilla.wagTail(); //nothing
          
          //change gorilla's tail wagging implementation at runtime:
          gorilla.setTailWagging(new HappyTailWagging());
          
          gorilla.wagTail(); //Result: Wag wag wag. I'm happy.
     }
}
{% endhighlight %}


### Conclusion

Besides flexibility you also accomplish better testability since the classes are loosely coupled. It's also possible to change behavior at runtime (with the setTailWagging method).