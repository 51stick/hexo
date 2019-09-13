---
title: Guava Collections Cookbook
date: 2017-04-30 19:20:14
categories: guava
tags: guava
---

This cookbook article is organized into small and focused recipes and code snippets for using Guava style collections.
<!-- more -->

### 1. Introduction

This cookbook article is organized into small and focused recipes and code snippets for using Guava style collections.

The format is that of a growing list of code examples with no additional explanation necessary – it is meant to keep common usages of the API easy to access during development.

[Guava Collections Cookbook]: http://www.baeldung.com/guava-collections	"Guava Functional Cookbook"

### 2. The Recipes

#### 2.1 downcast a List<Parent> to a List<Child>
> note: this is a workaround for non-covariant generified collections in Java
```java
class CastFunction<F, T extends F> implements Function<F, T> {
    @Override
    public final T apply(final F from) {
        return (T) from;
    }
}

List<TypeParent> originalList = Lists.newArrayList();
List<TypeChild> theList = Lists.transform(originalList, 
    new CastFunction<TypeParent, TypeChild>());
```
simpler alternative without Guava – involving 2 cast operations
```java
List<Number> originalList = Lists.newArrayList();
List<Integer> theList = (List<Integer>) (List<? extends Number>) originalList;
```

#### 2.2 adding an iterable to a collection
```java
Iterable<String> iter = Lists.newArrayList();
Collection<String> collector = Lists.newArrayList();
Iterables.addAll(collector, iter);
```

#### 2.3 check if collection contains element(s) according to a custom matching rule
```java
Iterable<String> theCollection = Lists.newArrayList("a", "bc", "def");
    boolean contains = Iterables.any(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
});
assertTrue(contains);
```

#### 2.4 alternative solution using search
```java
Iterable<String> theCollection = Sets.newHashSet("a", "bc", "def");
boolean contains = Iterables.find(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
       return input.length() == 1;
    }
}) != null;
assertTrue(contains);
```

#### 2.5 alternative solution only applicable to Sets
```java
Set<String> theCollection = Sets.newHashSet("a", "bc", "def");
boolean contains = !Sets.filter(theCollection, new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
}).isEmpty();
assertTrue(contains);
```
`NoSuchElementException` on Iterables.find when nothing is found
```java
Iterable<String> theCollection = Sets.newHashSet("abcd", "efgh", "ijkl");
Predicate<String> inputOfLengthOne = new Predicate<String>() {
    @Override
    public boolean apply(final String input) {
        return input.length() == 1;
    }
};
String found = Iterables.find(theCollection, inputOfLengthOne);
```
> this will throw the NoSuchElementException exception:
```java
java.util.NoSuchElementException
    at com.google.common.collect.AbstractIterator.next(AbstractIterator.java:154)
    at com.google.common.collect.Iterators.find(Iterators.java:712)
    at com.google.common.collect.Iterables.find(Iterables.java:643)
```
> solution: there is an overloaded find method that takes the default return value as an argument and can be called with null for the desired behavior:
```java
String found = Iterables.find(theCollection, inputOfLengthOne, null);
```

#### 2.6 remove all null values from a collection
```java
List<String> values = Lists.newArrayList("a", null, "b", "c");
Iterable<String> withoutNulls = Iterables.filter(values, Predicates.notNull());
```

#### 2.7 create immutable List/Set/Map directly
```java
ImmutableList<String> immutableList = ImmutableList.of("a", "b", "c");
ImmutableSet<String> immutableSet = ImmutableSet.of("a", "b", "c");
ImmutableMap<String, String> imuttableMap = 
    ImmutableMap.of("k1", "v1", "k2", "v2", "k3", "v3");
```

#### 2.8 create immutable List/Set/Map from a standard collection
``` java
List<String> muttableList = Lists.newArrayList();
ImmutableList<String> immutableList = ImmutableList.copyOf(muttableList);
 
Set<String> muttableSet = Sets.newHashSet();
ImmutableSet<String> immutableSet = ImmutableSet.copyOf(muttableSet);
 
Map<String, String> muttableMap = Maps.newHashMap();
ImmutableMap<String, String> imuttableMap = ImmutableMap.copyOf(muttableMap);
```

#### 2.10 alternative solution using builders
```java
List<String> muttableList = Lists.newArrayList();
ImmutableList<String> immutableList = 
    ImmutableList.<String> builder().addAll(muttableList).build();
 
Set<String> muttableSet = Sets.newHashSet();
ImmutableSet<String> immutableSet = 
    ImmutableSet.<String> builder().addAll(muttableSet).build();
 
Map<String, String> muttableMap = Maps.newHashMap();
ImmutableMap<String, String> imuttableMap = 
    ImmutableMap.<String, String> builder().putAll(muttableMap).build();
```