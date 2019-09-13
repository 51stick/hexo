---
title: Partition a List in Java
date: 2017-05-13 19:20:14
categories: guava
tags: guava
---

In this tutorial I will illustrate how to split a List into several sublists of a given size.
<!-- more -->

### 1. Overview
In this tutorial I will illustrate how to split a List into several sublists of a given size.

For a relatively simple operation, there is surprisingly no support in the standard Java collection APIs. Luckily, both Guava and the Apache Commons Collections have implemented the operation in a similar way.

This article is part of the “Java – Back to Basic” series here on Baeldung.

[Partition a List in Java]: http://www.baeldung.com/java-list-split	"Partition a List in Java"

### 2. Use Guava to partition the List
Guava facilitates partitioning the List into sublists of a specified size – via the Lists.partition operation:
```java
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

#### 2.1 Use Guava to partition a Collection
Partitioning a Collection is also possible with Guava:
```java
@Test
public void givenCollection_whenParitioningIntoNSublists_thenCorrect() {
    Collection<Integer> intCollection = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
 
    Iterable<List<Integer>> subSets = Iterables.partition(intCollection, 3);
 
    List<Integer> firstPartition = subSets.iterator().next();
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(1, 2, 3);
    assertThat(firstPartition, equalTo(expectedLastPartition));
}
```
Keep in mind that the partitions are sublist views of the original collection – which means that changes in the original collection will be reflected in the partitions:
```java
@Test
public void givenListPartitioned_whenOriginalListIsModified_thenPartitionsChangeAsWell() {
    // Given
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);
 
    // When
    intList.add(9);
 
    // Then
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8, 9);
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

### 3. Use Apache Commons Collections to partition the List
The latest releases of Apache Commons Collections have recently added support for partitioning a List as well:
```java
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = ListUtils.partition(intList, 3);
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```
There is no corresponding option to partition a raw Collection – similar to the Guava Iterables.partition in Commons Collections.

Finally, the same caveat applies here as well – the resulting partition are views of the original List.

### 3. Use Java8 to partition the List
### 3.1 Collectors partitioningBy
We can use Collectors.partitioningBy() to split the list into 2 sublists – as follows:
```java
@Test
public void givenList_whenParitioningIntoSublistsUsingPartitionBy_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
 
    Map<Boolean, List<Integer>> groups = 
      intList.stream().collect(Collectors.partitioningBy(s -> s > 6));
    List<List<Integer>> subSets = new ArrayList<List<Integer>>(groups.values());
 
    List<Integer> lastPartition = subSets.get(1);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(2));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```
Note: The resulting partitions are not a view of the main List, so any changes happen to the main List won’t affect the partitions.

#### 3.2 Collectors groupingBy
We also can use Collectors.groupingBy() to split our list to multiple partitions:
```java
@Test
public final void givenList_whenParitioningIntoNSublistsUsingGroupingBy_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
 
    Map<Integer, List<Integer>> groups = 
      intList.stream().collect(Collectors.groupingBy(s -> (s - 1) / 3));
    List<List<Integer>> subSets = new ArrayList<List<Integer>>(groups.values());
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```
Note: Just as Collectors.partitioningBy() – the resulting partitions won’t be affected by changes in main List.

####3.3  Split the List by separator
We can also use Java8 to split our List by separator:
```java
@Test
public void givenList_whenSplittingBySeparator_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 0, 4, 5, 6, 0, 7, 8);
 
    int[] indexes = 
      Stream.of(IntStream.of(-1), IntStream.range(0, intList.size())
      .filter(i -> intList.get(i) == 0), IntStream.of(intList.size()))
      .flatMapToInt(s -> s).toArray();
    List<List<Integer>> subSets = 
      IntStream.range(0, indexes.length - 1)
               .mapToObj(i -> intList.subList(indexes[i] + 1, indexes[i + 1]))
               .collect(Collectors.toList());
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```
Note: We used “0” as separator – we first obtained the indices of all “0” elements in the List, then we split the List on these indices.

###4. Conclusion
The solutions presented here makes use of additional libraries – Guava or the Apache Commons Collections library. Both of these are very lightweight and extremely useful overall, so it makes perfect sense to have one of them on the classpath; however, if that is not an option – a Java only solution is shown here.

The implementation of all these examples and code snippets can be found in my github project – this is an Eclipse based project, so it should be easy to import and run as it is.
