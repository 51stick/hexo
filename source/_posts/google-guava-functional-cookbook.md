---
title: Guava Function Cookbook
date: 2017-05-08 19:20:14
categories: guava
tags: guava
---

This cookbook is organized into small and focused recipes and code snippets for using the Guava functional-style elements – Predicates and Functions.
<!-- more -->

### 1. Overview
This cookbook is organized into small and focused recipes and code snippets for using the Guava functional-style elements – Predicates and Functions.

The cookbook format is focused and practical – no extraneous details and explanations necessary.

[Guava Functional Cookbook]: http://www.baeldung.com/guava-functions-predicates	"Guava Functional Cookbook"

### 2. The Cookbook
#### 2.1 filter a collection by a condition (custom Predicate)
```java
List<Integer> numbers = Lists.newArrayList(1, 2, 3, 6, 10, 34, 57, 89);
Predicate<Integer> acceptEven = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
List<Integer> evenNumbers = Lists.newArrayList(Collections2.filter(numbers, acceptEven));
Integer found = Collections.binarySearch(evenNumbers, 57);
assertThat(found, lessThan(0));
```

#### 2.2 filter out nulls from a collection
```java
List<String> withNulls = Lists.newArrayList("a", "bc", null, "def");
Iterable<String> withoutNuls = Iterables.filter(withNulls, Predicates.notNull());
assertTrue(Iterables.all(withoutNuls, Predicates.notNull()));
```

#### 2.3 check condition for all elements of a collection
```java
List<Integer> evenNumbers = Lists.newArrayList(2, 6, 8, 10, 34, 90);
Predicate<Integer> acceptEven = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
boolean result = Iterables.all(evenNumbers, acceptEven);
```

#### 2.4 negate a predicate
```java
List<Integer> evenNumbers = Lists.newArrayList(2, 6, 8, 10, 34, 90);
Predicate<Integer> acceptOdd = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) != 0;
    }
};
assertTrue(Iterables.all(evenNumbers, Predicates.not(acceptOdd)));
```

#### 2.5 apply a simple function
```java
List<Integer> numbers = Lists.newArrayList(1, 2, 3);
List<String> asStrings = Lists.transform(numbers, Functions.toStringFunction());
assertThat(asStrings, contains("1", "2", "3"));
```

#### 2.6 sort collection by first applying an intermediary function
```java
List<Integer> numbers = Arrays.asList(2, 1, 11, 100, 8, 14);
Ordering<Object> ordering = Ordering.natural().onResultOf(Functions.toStringFunction());
List<Integer> inAlphabeticalOrder = ordering.sortedCopy(numbers);
List<Integer> correctAlphabeticalOrder = Lists.newArrayList(1, 100, 11, 14, 2, 8);
assertThat(correctAlphabeticalOrder, equalTo(inAlphabeticalOrder));
```

#### 2.7 complex example – chaining predicates and functions
```java
List<Integer> numbers = Arrays.asList(2, 1, 11, 100, 8, 14);
Predicate<Integer> acceptEvenNumber = new Predicate<Integer>() {
    @Override
    public boolean apply(Integer number) {
        return (number % 2) == 0;
    }
};
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};
 
FluentIterable<Integer> powerOfTwoOnlyForEvenNumbers = 
FluentIterable.from(numbers).filter(acceptEvenNumber).transform(powerOfTwo);
assertThat(powerOfTwoOnlyForEvenNumbers, contains(4, 10000, 64, 196));
```

#### 2.8 compose two functions
```java
List<Integer> numbers = Arrays.asList(2, 3);
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};
List<Integer> result = Lists.transform(numbers, 
    Functions.compose(powerOfTwo, powerOfTwo));
assertThat(result, contains(16, 81));
```

#### 2.9 create a Map backed by a Set and a Function
```java
Function<Integer, Integer> powerOfTwo = new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer input) {
        return (int) Math.pow(input, 2);
    }
};
Set<Integer> lowNumbers = Sets.newHashSet(2, 3, 4);
 
Map<Integer, Integer> numberToPowerOfTwoMuttable = Maps.asMap(lowNumbers, powerOfTwo);
Map<Integer, Integer> numberToPowerOfTwoImuttable = Maps.toMap(lowNumbers, powerOfTwo);
assertThat(numberToPowerOfTwoMuttable.get(2), equalTo(4));
assertThat(numberToPowerOfTwoImuttable.get(2), equalTo(4));
```