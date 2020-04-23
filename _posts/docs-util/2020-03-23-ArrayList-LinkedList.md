---
title: Java中ArrayList和LinkedList的性能分析
permalink: /docs-util/ArrayList/LinkedList
tags: CodeMark
key: docs-util-ArrayList-LinkedList
---
 *ArrayList* 和 *ArrayList* 是Java集合框架中经常使用的类。如果你只知道从基本性能比较 *ArrayList* 和 *ArrayList* ，那么请仔细阅读这篇文章。
 *ArrayList* 应该在需要更多搜索操作的地方使用，并且 *ArrayList* 应该在需要更多插入和删除操作的地方使用。

 *ArrayList* 使用 Array 数据结构， *ArrayList* 使用  *DoublyArrayList*  数据结构。在这里，我们要讨论基础数据结构如何影响插入，搜索的性能，以及删除操作  *ArrayList*  和  *ArrayList* 。下面是使用 *ArrayList* 和 *LinkedList* 不同操作的示例:下面我们将比较 *ArrayList* 和 *LinkedList* 的操作，看哪一个在性能方面更有效。

## 在最后索引处插入值（块1和2）

当我们在最后一个索引处插入一个值时， *ArrayList* 必须 ：

- 检查基础数组是否已满。

- 如果数组已满，则将数据从旧数组复制到新数组（大小比旧数组大两倍），

- 然后在最后一个索引处添加值。

另一方面， *LinkedList* 只是在底层 *DoublyLinkedList* 的尾部添加该值。

两者都具有时间复杂度O（1），但是由于在 *ArrayList* 中创建新数组的附加步骤，其最坏情况复杂度达到N的顺序，这就是为什么我们更喜欢使用 *LinkedList* .

## 在给定指数处插入值（第3和第4块）

当我们在给定索引处插入值时，ArrayList必须 :

- 检查基础数组是否已满。

- 如果数组已满，则将数据从旧数组复制到新数组（大小为旧数组的两倍）。

- 之后，从给定索引开始，将值移动一个索引以为新值创建空间。

- 然后在给定索引处添加值。

另一方面， *LinkedList* 只是通过重新排列底层 *DoublyLinkedList* 的指针来查找索引并在给定索引处添加该值。

两者都具有时间复杂度O（N），但是由于在 *ArrayList* 中创建新数组并将现有值复制到新索引的附加步骤，我们更喜欢使用LinkedList.

## 按值搜索（第5和第6块）

当我们在 *ArrayList* 或LinkedList中搜索任何值时，我们必须遍历所有元素。该操作具有O（N）时间复杂度。看起来像 *ArrayList* 和LinkedList都具有相同的性能。

这里我们需要注意的是，数组（ *ArrayList* 的底层数据结构）将所有值存储在连续的内存位置，但是 *DoublyArrayList* 将每个节点存储在一个随机的内存位置。迭代连续内存位置比随机内存位置更具性能效率，这就是为什么我们在按值搜索时更喜欢使用 *ArrayList* 而不是 *ArrayList* 。

## 按索引获取元素（第7和第8块）

当我们通过Index获得元素时， *ArrayList* 是一个明显效果更好。

 *ArrayList* 可以为您提供O（1）复杂度的任何元素，因为该数组具有随机访问属性。您可以直接访问任何索引而无需遍历整个数组。

 *ArrayList* 具有顺序访问属性。它需要迭代每个元素以达到给定的索引，因此从 *ArrayList* 通过索引获取值的时间复杂度是O（N）。

## 按值删除（块9和10）

它类似于在给定索引处添加值。要在 *ArrayList* 和 *ArrayList* 中按值删除元素，我们需要迭代每个元素以到达该索引，然后删除该值。该操作具有O（N）复杂度。

不同的是，要从 *ArrayList* 中删除一个元素，我们只需要修改指针，但是在 *ArrayList* 中，我们需要在删除值的索引之后移动所有元素以填充创建的间隙。

由于移位是昂贵的操作然后修改指针，因此即使在相同的总体复杂度O（N）之后，我们更喜欢 *ArrayList* ，其中需要更多按值操作删除。

## 按索引删除（第11和12栏）

为了通过索引删除， *ArrayList* 使用O（1）复杂度中的随机访问来查找该索引，但是在移除元素之后，移位其余元素会导致整体O（N）时间复杂度。

另一方面， *ArrayList* 花费O（N）时间来使用顺序访问来查找索引，但是要删除元素，我们只需要修改指针。

由于移位是比迭代更昂贵的操作，因此如果我们想要逐个删除元素， *ArrayList* 会更有效。
