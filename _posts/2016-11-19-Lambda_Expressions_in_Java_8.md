---
layout: post
title: Lambda Expression in Java 8
---

The Lambda expression is a new feature added in Java 8, which significanly enhances the expressive power of the language. It's even compared to the addition of generics to emphasise it's importance. Indeed, together with functional interfaces, method references and streams, we could write more productive and elegant code than ever before. There are loads of tutorials out there on how to use lambda expressions, therefore I just skip this part and focus more on things that I found interesting.

### The Evolution of Expressive Power
Suppose you have a list of strings and you simply want them to be ordered by their sizes such that shorter strings come before longer ones. Here is a simplified Comparator interface.
```java
interface Comparator<T> {
    int compare(T o1, T o2);
}
```
A naive try without using an anonymous class and lambda expression would be:
```java
class myComparator<String> implements Comparator<String> {
    public int compare(String a, String b) {
        return a.length() < b.length() ? -1 : a.length() == b.length() ? 0 : 1;
    }
}
    ... 
// where names is a list of strings
Collections.sort(names, new myComparator());
    ...
```
The problem here is it is too verbose. In order to do this simple operation, you need to declare your own implementing class and often give it a dummy name. Also, mostly, the declaration is far away from its usage which adds extra hassles while reading code. You would probably find youself spending much time in context switching, scanning for certain declarations etc, whereas it could have been coded in a nicer way. Now, the anonymous class comes to alleviate the pain. The above code can then be refactored as:
```java
// where names is a list of strings
Collections.sort(names, new Comparator<String> {
    public int compare(String a, String b) {
        return a.length() < b.length() ? -1 : a.length() == b.length() ? 0 : 1;
    }
});
```
Now the code above looks more compact due to the fact that both the declaration and its usage are in the same place, and most importantly you don't need to force yourself to think of a dummy name (phew!). However, from the readability point of view, it still is not good enough. To specify a simple behaviour, you'd have to include the bulk of code as if you were writing a separate class declaration. The bulkiness serves nothing but for the benefit of the language completeness. What we want to stress from above code is actually a fuction, a behaviour which idealy could be passed around. Now, the lambda expression comes to rescue and we can rewrite the code as:
```java
 // where names is a list of strings
 // the type of a and b is inferred by the compiler
 
Collections.sort(
    names, 
    (a, b) -> a.length() < b.length() ? -1 : a.length() == b.length() ? 0 : 1
)
``` 
As you can see, this code is way more succinct than the previous one. The lambda expression really shifts your focus from writing code appealing to the compiler to appealing to humans. You can now think in a more abstract way, just treat it as passing a function, a behaviour around, without worrying too much about the details, because a compiler has taken care of it.

With the arrial of lambda expressions, a large number of Java APIs have been enriched to help us think in **Lambda**. For example, the Comparator interface (as of Java 8, an interface can have static methods and default instance methods) has a bunch of static methods and default methods with functional interfaces as parameters to assist us deal with common cases, therefore, the code can be further simplified as:
```java
Collections.sort(
    names, 
    Comparator.comparingInt(String::length)
)
``` 

### Variable Capture
Similar to a local class, in a lambda expression you can access static variables and instance variables of the enclosing class, and local variables of the enclosing method. In terms of using local variables from the enclosing mothod, this specifc situation is referred to as a variable capture. Because a local variable is situated on a stack, which means its value will become invalid once its enclosing method exits, and the local variable has to survive the whole lambda execution which is normally longer than the life time of its enclosing method, a local variable then must be captured. Once captured, when does Java update its value if the local variable in the enclosing scope has been changed? The answer is simply no, you can't even change the value. There is a restriction on captured variables: they should be declared final or effectively final (effectively final means the value of a local variable is not changed once initialised). Had this restriction been lifted, there would likely to be confusion caused (which value to use, the one when the lambda expression is defined or the updated value) and extra complexity in concurrence. This rule may sound limiting, but mostly it is enough for normal uses, because we still can change underlying object even if the variable pointing to it is final or effectively final. Like the code shown below, a lambda expression is used to collect strings with size longer than a number into a list:
```java
List<String> getStringsLongerThan(List<Strings> strings, int len) {
    List<String> res = new ArrayList<>();
    strings.stream()
           .filter(s -> s.length() > len)
           .forEach(res::add)
    return res;
}

// this is for demonstration purpose, a more appropriate way should be  
// return strings.stream().filter((s -> s.length() > len).collect(Collectors.toList())
```
#### Quiz?
**Q:** In a lambda expression, if you access a instance int variable of the enclosing class, can you change the value in or outside your lambda?
**A:** Yes, you can. Although the same final or effective final rule applies in this case, in a lambda expression a compiler generates a implicit final reference **this** which refers to the enclosing class.(which you can think of as captureing the final **this** reference) Since it's done for you behind scene, you don't normally feel this rule when using instance variables.

### Anonymous Class
How lambdas are actually implemented behind the scene? Reference 1 gives details on how lambda expressions are translated. In a nutshell, a lambda expression is converted to the instance of its functional interface. In a way, it is very like an anonymous class, for its anonymity and its funcitona body providing concrete implementation for its target type in the functional interface; however, there are differences. Since lambda expressions are lexically scoped, they don't introduce an extra level of scope and therefore they don't have the shadowing problem (a variable shadows the other one with the same name in the outer scope) and they don't have **this** referring to themselves. Every thing in a lambda appears as if they are in the enclosing context; therfore, **this** refers to the enclosing instance and any protential shadowing would pose variable is already defined error.  

### Reference
1. [Translation of Lambda Expressions](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html)  
2. Chapter 14: Lambda Expressions and Method References; Java A Beginner's Guide, 6th Edition  
3. Chapter 3: Lambda expressions; Java 8 in Action: Lambdas, streams, and functional-sytle programming  
4. [Lambda Expressions; The Java Tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)  
5. [The Java Language Specification, Java SE 8 Edition](https://docs.oracle.com/javase/specs/jls/se8/jls8.pdf)  

