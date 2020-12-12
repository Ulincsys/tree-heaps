# Tree Heaps - A Case For Tree Based  Binary Heaps
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
		B				C
	D		E		F		G
H
```

Do note that these statements make no claim as to whether or not a given "node" is initialized. That task is left as an exercise to the reader.

### Generating a heap from a given dataset

```c
void heapify(Array arr, int parent, int arraySize) {
	int left = 2 * parent;
	int right = (2 * parent) + 1;

	int hasleft = left < arraySize;
	int hasright = right < arraySize;

	if(hasleft) {
		if(hasright) {
			if((arr[right] > arr[left]) && (arr[right] > arr[parent])) {
				int temp = arr[right];
				arr[right] = arr[parent];
				arr[parent] = temp;
				heapify(arr, right, arraySize);
				return;
			}
		}
		if(arr[left] > arr[parent]) {
			int temp = arr[left];
			arr[left] = arr[parent];
			arr[parent] = temp;
			heapify(arr, left, arraySize);
			return;
		}
	}
}
```