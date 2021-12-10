# Data Structures and Algorithams 

<hr />

## Algorithams 
**An algorithm** -  Is a set of well-defined instructions to solve a particular problem. 
It takes a set of input and produces a desired output

- Input
- Output
- Definiteness 	- It must be clear
- Finiteness 	- It must stop
- Effectiveness - some purpose

**How to Analyse Alogithm**

1. Time  -  must be Time efficient
2. Space - must be memory effficient
3. N/W 	 -  
4. Power - 
5. CPU Registers - 

Every statement/line is 1 unit of time

**Frequency Count Method**
- Means, the number of times the statement is executed in the program

- To calculate the time, consider the degree
    ```
	- sum(a,b) 		    => f(n) = 2n +3                  => O(n)
	- add(a,b,n)        => f(n) = 2n2 + 2n + 1           => O(n2)
	- multiply(a,b,n)   => f(n) = 3n3 + 3n2 + 2n + 1    => O(n3)
    ```

- To Calculate space complexity, consider all variables
    ```
	- sum(a,b)		    => s(n) = n+3        => O(n)
 	- add(a,b,n)		=> s(n) = 3n2 + 3    => O(n2)
 	- multiply(a,b,n)	=> s(n) = 3n2 +4     => O(n2)
    ```

**Time Complexity Examples**

```
- for(i=0;i<n;i++) 		        => O(n)
- for(i=n;i>0;i--) 		        => O(n)
- for(i=0;i<n;i=i+2)		n/2 => O(n)
- for(i=0;i<n;i++)	
    for(j=0;j<i;j++)	        => O(n^2)
- for(i=1;i<n; i=i++) 
  for(j=0;j<n;j++)	            =>O(n^2)
- p=0
  for(i=0;p<=n;i++) 		    =>O(root N)

- for(i=1;i<n; i=i*2)	        => O(log n base 2)
- for(i=1;i<n; i=i*3)           => O(log n base 3)
- for(i=n;i>1; i=i/2)           => O(log n base 2 )

- for(i=0;i*i<n;i++) 	        => O(root n)

- for(i=0;i<n;i++) 	
    stmt
  for(j=0;j<i;j++)	
    stmt                        => O(n)
- p=0
  for(i=0;i<n;i*2) 	
    stmt
  for(j=0;j<p;j*2)	
	stmt                        => O(log log n)

- for(i=0;i<n;i++) 	
    stmt
  for(j=0;j<i;j=n*2)	
	stmt                        =>O(nlogn)
```
**Big O noation is for...**
- Time Complexity (how much runtime it will take)
- Space Complexity (how much memory it will take)

**Types of Time Complexities (Classes of Funcitons)**
- Constant Time Complexity 		-  O(1)
- Linear Time Complexity 		-  O(n)
- Logarithmic Time  Complexity 	-  O(log n)
- Quadratic Time Complexity 	- O(n ** 2)
- Cubic Time Complexity			- O(n ** 3)
- Exponential Complexity		- O(2 power n)

**How to calculate Time Complexities**
- Mathematical way
- Programatic or Practical way
  1. Find the Fast increasing term
  2. Remove the cofficient in that term
   
**Compare Class of Functions**
```
1 < log n < /n < n < n log n < n2 < n3 ... 2n < 3n < Nn < O(1) < O(logN) < O(N) < O(NlogN) < O(N^2) < O(2^N)< O(N!)
```

**Log Values**
* Logn n = 1
* logn 1 = 0

```  
log1=0 	1	1	2
log2=1	2 	4	4
log4=2 	4 	16 	16
log8=3 	8 	64 	256
log16=4 16 	256 65536
```

**Asymptotic Notations - Big Oh**
1. BigO  	Upper bound		(f(n) <= BigO(n) )
2. Omega  	Lower bound 	(f(n) >= Omega(g(n) )
3. Theta 	Average bound 	(f(n) = Theta(n) )

* BigO or Omega is when we are not sure
* Theta gives best 
* IF Theta can't find, go for BigO or Omega

<hr />

## Data Structures
Data Structures are way storing and organizing data in a computer for efficient access and modification

### Array
* Linear Data type; in Python using List
* Similar data types - Stores  data as contiguous memory locations

### Linked List
* Linear Data type, in Python using List
* Stores as individual nodes and has pointers to next node
 * **Singly-linked list**: 
   * linked list in which each node points to the next node  
   * the last node points to null
 * **Doubly-linked list**: 
   * linked list in which each node has two pointers, p and n, such that p points to the previous node and n points to the next node;
   * the last node's n pointer points to null
 * **Circular-linked list**: 
   * linked list in which each node points to the next node  
   * the last node points back to the first node
 * **Time Complexity**:
    * Access: O(n)
    * Search: O(n)
    * Insert: O(1)
    * Remove: O(1)
* **Diff**
  - For LL No need to allocate momory
  - LL Insertion is easy
  - LL Memory is higher (p & n)
  - Insertion/Delete at begining = O(1)
  - Insert/Delete at end = O(n)

### Stack
* Last In First Out operation; 
  * push - Insert at end
  * pop - remove at end
* In python using List/collections.deque/queue.LifoQueue
* Time Complexity
  * Push: Insert - O(1)
  * Pop: Delete	- O(1)
  * Search 		- O(n)

### Queue
* First In First Out operation; 
  * push - Insert at end
  * pop - remove at start
* In python using List/collections.deque/queue.LifoQueue
* Time Complexity
  * Push: Insert - O(1)
  * Pop: Delete	- O(1)
  * Search 		- O(n)
  
### Hashmap/Hashtable
* **Hashing** 
  * assigning an object into a unique index called as  key. 
  * Object is identified using a key-value pair 
  * Dictionary - collection of objects
* **Hash table** storing elements as key-value pairs, identify using key
* Hash table memory efficient
* Lookp/Insertion/Deletion Complexity is using key always O(1)
* **Collision:** if hash function generetes same function the conflict called Collision.
  * **Chaining** - one key, store values as List in dictionary
  * **Open Addressing** - Linear/Quadratic Probing and Double Hashing
    * **Linear Probing** - collision is resolved by checking the next slot
    * **Quadratic Probing**- similar to linear probing but the spacing between the slots is increased
    * **Double hashing** - applying a hash function & nother hash function
* Time Complexity:
  * Lookp/Insertion/Deletion is using key always O(1)
* ### Tree
  * A Tree is nonlinear hierarchical, undirected, connected, acyclic graph
    * Root - top most node of a tree.
    * Node - data and pointers to another node
    * Edge - link between nodes
    * leaf nodes(L1, L2, L3)
  * **Binary Tree**
    * parent node can have at most two children
    * Left node has lesser numbers
    * Right node has greater numbers
    * **Full Tree** - a tree in which every node has either 0 or 2 children
    * **Perfect Binary Tree** - binary tree with exactly 2 child nodes at all level
    * **Complete Tree** -  every level must be filled, leaf nodes lean towards left
    * **Pathological Tree** - tree having a single child either left or right
    * **Skewed Binary Tree** - tree is either dominated by the left Pathological nodes or the right Pathological nodes
    * **Balanced Binary Tree** - difference between the height of the left and the right is either 0 or 1
  * **Binary Search Tree**
    * Left subtree has lesser numbers,
    * Right subtree has greater numbers
    * BST every node has max 2 nodes  
    * Every Iteration we reduce space by half(1/2))
    * if n=8  & 3 iterations, log 8 = 3 => O(log(n))
    * Access, Insert, Search adn Delete  = Θ(log(n))
    * **Breadth first search**
      * level1, level2(left to right), leve3(left to right)
    * **Depth first search**
      * In order traversal (left, root, right)
      * Pre order traversal (root, left and right)
      * Post order traversal (left, right and root)
    * **Deleting a node**
      * delete a node with no child
      * delete a node with one child
      * delete a node with multiple childs

### Graph