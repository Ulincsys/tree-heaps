# Tree Backed Heaps - A Case For Linked Binary Heaps
Many heap algorithms utilize pseudo-trees backed by arrays for implementing heap algorithms. This approach is useful from a performance perspective because it eliminates the need to maintain a linked data structure, and also reduces the need for edge cases in insertion and deletion.

In this document we will first analyze the array implementation and the associated algorithms for insertion and deletion which guarantee balance in the pseudo-tree, after which we will introduce the case for a tree-based heap and analyze the algorithms required to maintain the linked structure.

This document assumes you have a precursory understanding of Binary Trees, heaps, basic comparison logic and basic array logic. If not, it is recommended that you familiarize yourself with these concepts in order to gain the most from reading further.

* [Relational Operators on Wikipedia](https://en.wikipedia.org/wiki/Relational_operator)
* [Arrays on Wikipedia](https://en.wikipedia.org/wiki/Array_data_structure)
* [Binary Trees on Wikipedia](https://en.wikipedia.org/wiki/Binary_tree)
* [Heaps on Wikipedia](https://en.wikipedia.org/wiki/Heap_(data_structure))

## Array based binary heaps
Array backed heaps utilize the instantaneous access property of arrays to reduce the heap building process to two simple base cases, bounded by the size of the container. These implicit data structures have very low overhead compared to linked structures, and similarly have favorable time-complexity constants.

### A generalization of array-backed binary trees
In a pseudo-tree backed by an indexed data structure (IE: an array, table, matrix etc...), an algorithm can mathematically define the tree structure in terms of the size of the container.

Generally, every element at index ` i ` is a tree node with at most two children. The "left" child of a given node is defined as the element at

	L = (i * 2)
and the "right" child is defined as the element at

	R = (i * 2) + 1
Unlike in a linked tree, the number of children a given node has is based upon the size of the container. Given a 1-indexed data container with maximum size `arraySize`:

* if `L < arraySize`, then the node has two children in the tree
* else if `L == arraySize`, then the node has only a "left" child
* else the node has no children in the tree (because `(i * 2) > arraySize`

Given this, we can thus define that every "node" at an index greater than `(arraySize / 2)` is a leaf node, because every value of `(i * 2)` will be greater than the size of the container. This holds true to the fact that half of the nodes in a balanced binary tree will be leaf nodes, as each row of a binary tree R<sub>0</sub> ... R<sub>n - 1</sub> will have at most  n<sup>R</sup> nodes. Therefore, to calculate the number of rows that will be present in the binary tree representation, we must consider the base-2 logarithm of the container size. Specifically, the number of rows in the tree representation will be **floor(log<sub>2</sub>(arraySize)) + 1**. As an example:

	array = { A, B, C, D, E, F, G, H }

There are 8 elements in this container, therefore the number of rows in this tree will be **floor(log<sub>2</sub>(8)) + 1 = 4**. Indeed, if we visualize the tree representation of this dataset, we see there are exactly four rows:
```
				A
			/		\
		B				C
	/		\		/		\
	D		E		F		G
/
H
```

Do note that these statements make no claim as to whether or not a given "node" is initialized. That task is left as an exercise to the reader.

### Generating a heap from a given dataset
Now that we have defined the structure of an array-backed binary tree, let's analyze an algorithm to build a heap using this structure.

Let us first consider the simplest base case for building a heap. Given the following complete binary tree:
```
array = { 3, 2, 1 }
-------------------
		3
	/		\
	2		1
```

Assume that we are building a min-heap in this hypothetical. This tree does not satisfy the heap condition because the node at the root is greater than at least one of its children. In order to rectify this, we must swap the node with the smallest key into the root of the tree, thus it is guaranteed to satisfy the heap condition.

Let's see that algorithm represented in pseudocode:
```python
if (node[left] < node[right]) AND (node[left] < node):
	swap node, node[left]
else if (node[right] < node[left]) AND (node[right] < node):
	swap node, node[right]
```

You can see that after this algorithm is executed, the resulting tree will satisfy the heap condition.

```
array = { 1, 2, 3 }
-------------------
		1
	/		\
	2		3
```

Now, to generalize for any tree, the goal is to repeat this algorithm for each child of every node in the tree. Assume that we are given a complete dataset with *n* elements that we would like to build into a heap. In order to guarantee that every subtree in the dataset satisfies the heap condition, we must visit every node that has at least one child. We can ensure that we visit every node with at least one child by starting with the node at index `(arraySize / 2)` and running the algorithm backwards from there down to the first node.

Many descriptions of this algorithm call it "heapify", which is itself a description of how it manipulates the dataset. The algorithm comes in two parts, let's look at an example written in C:

> The type of the array is not particularly relevant to the example, but in the case of the following code, the type Array is defined as `typedef int* Array;`

The first part of the algorithm is an iterative function *buildHeap()* which calls *heapify()*
```c
void buildHeap(Array arr, arraySize) {
	int start = (arraySize / 2);
	while(--start >= 0) {
		heapify(arr, start, arraySize);
	}
}
```

As you can see, our *buildHeap()* function calls *heapify()* `(arraySize / 2)` times, and runs backwards with respect to the order of the indices. This guarantees that we will visit each node which has children at least once, and that every subtree will satisfy the heap condition. Now, the second part of the algorithm is a recursive function *heapify()* which does the work of comparing the children to each other and the parent node, and re-ordering all of the children if needed.

Do note that in C arrays are 0-indexed as opposed to 1-indexed. This is not particularly an issue for our algorithm, but it does require us to modify the definition of how we locate each node's children. Simple offset the results by 1 as shown below, and the definition should again be correct.

As you can see, for every time we call the *heapify()* subroutine, we ensure that the heap condition is satisfied in all of the subtrees by recursively calling *heapify()* if we manipulate either child.

```c
void heapify(Array arr, int parent, int arraySize) {
	int left = 2 * parent + 1;
	int right = 2 * parent + 2;

	if(left < arraySize) { // This node has at least one child
		if(right < arraySize) { // This node has two children
			if((arr[right] < arr[left]) && (arr[right] < arr[parent])) {
				int temp = arr[right];
				arr[right] = arr[parent];
				arr[parent] = temp;
				heapify(arr, right, arraySize);
				return;
			}
		}
		if(arr[left] < arr[parent]) {
			int temp = arr[left];
			arr[left] = arr[parent];
			arr[parent] = temp;
			heapify(arr, left, arraySize); 
			return;
		}
	}
}
```

At the conclusion of the call to *buildHeap()*, the data in the array will be organized in such a way that the implicit binary tree satisfies the heap condition (min-heap in this case).

## Tree based binary heaps