---
published: true
title: The Time I Found a bug in Java Lists
layout: post
summary: Bug in Java lists allows removing from lists
excerpt_separator: <!--more-->
author: Val
category: Java
tags:
- java
- bugs
---
So I have a funny story!  I was teaching some Java to one of my cousins, who is in college for CS, and we had just gotten to talking about iterators; specifically, why you shouldn't try to remove an element from a collection while you're actively iterating over that collection.  "If you try to do that," I told him confidently, "Java will slap you on the wrist with a ConcurrentModificationException!"

Then, to demonstrate, I asked him to go ahead and write some code that did just that - and sure enough, there was no ConcurrentModificationException, and the Java runtime cheerfully obliged by removing an item right from the ArrayList we were iterating over!

Digging deeper, we found what looks like a loophole in the implementation of ArrayList as well as LinkedList.
<!--more-->

For starters, the code my cousin wrote looked something like the snippet below; also put the code up here: https://github.com/valakkapeddi/list_remove_bug.
```java
        private final List<String> arrayList = new ArrayList<>();

        arrayList.add("a");
        arrayList.add("b");
        arrayList.add("c");
        arrayList.add("d");
        arrayList.add("e");
        arrayList.add("f");


        for(String each: arrayList) {
            System.out.println(each);
            if(each.equals("e")) {
                System.out.println("Removing " + each);
                arrayList.remove(each);
            }
        }

         arrayList.stream().forEach(System.out::println);
```
If you try to remove an earlier element (such as "a"-"d"), the ConcurrentModificationException gets raised as excepted.  So why does this happen?  It turns out that the check for comodification happens in the call to next(). When you remove either the last or next-to-last element, you hit the loophole; the Java runtime uses hasNext() to decide whether to call next() to try and retrieve the next element. hasNext() makes this determination by comparing the size of the list against the 'index' of the current element.  Since you've removed an element from the list, thereby reducing the size of the collection by one, the hasNext() check is cheated.  Java never calls next() for the "true" last element, and therefore doesn't check for comodification.

It seems like this could be fixed if we checked for comodification in hasNext() - I wonder what the implications of that would be, other than the performance hit?
