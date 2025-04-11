---
title: Binary Trees using C
date: 2025-04-11
categories: [DSA]
tags: [DSA, C, programming]     # TAG names should always be lowercase
author: Shibu
description: Diving Deep into Binary Trees using C
toc: true # default is true
pin: false
comments: true
---

## Diving Deep into Binary Trees: Concepts and C Implementation

Binary trees are fundamental data structures in computer science, forming the basis for more complex structures like heaps, binary search trees, and various balanced trees. Their hierarchical nature makes them incredibly efficient for certain types of operations, particularly searching and sorting.

In this post, we'll explore:

1.  **What a Binary Tree is** (Core Concepts & Terminology)
2.  **Why Use Binary Trees?** (Advantages & Applications)
3.  **Representing Binary Trees in C** (The `struct Node`)
4.  **Common Operations with C Implementations:**
    *   Creating a Node
    *   Insertion (specifically for Binary Search Trees - a common type)
    *   Traversals (Inorder, Preorder, Postorder)
    *   Searching (specifically for Binary Search Trees)
5.  **Putting it all Together** (A Sample `main` function)
6.  **Complexity Considerations**

Let's dive in!

### 1. What is a Binary Tree?

A **Binary Tree** is a hierarchical data structure where each node has *at most* two children, referred to as the **left child** and the **right child**.

**Key Terminology:**

*   **Node:** The basic unit containing data and pointers (references) to its children.
*   **Root:** The topmost node in the tree. A tree has only one root.
*   **Edge:** A link connecting a parent node to a child node.
*   **Parent:** A node that has at least one child node.
*   **Child:** A node that has a parent node.
*   **Leaf Node (or Terminal Node):** A node with no children.
*   **Internal Node:** A node with at least one child (i.e., not a leaf node).
*   **Height of a Node:** The number of edges on the longest path from the node down to a leaf node. The height of a leaf node is 0.
*   **Height of a Tree:** The height of the root node. An empty tree has a height of -1 (or sometimes 0, depending on convention).
*   **Depth of a Node:** The number of edges from the root node down to that node. The depth of the root is 0.
*   **Subtree:** A tree consisting of a node and all its descendants. Each node in a binary tree can be considered the root of a subtree.

**Visual Example:**

```
      ( Root: A )
       /       \
      /         \
( B: Left Child )  ( C: Right Child )
   /     \             \
  /       \             \
( D: Leaf ) ( E: Leaf )   ( F: Leaf )
```

*   **Root:** A
*   **Internal Nodes:** A, B, C
*   **Leaf Nodes:** D, E, F
*   **Parent of D & E:** B
*   **Left Child of A:** B
*   **Left Child of C:** None (NULL)
*   **Height of Node B:** 1
*   **Height of Tree:** 2 (longest path A -> B -> D or A -> B -> E)
*   **Depth of Node F:** 2

**Important Note:** This definition doesn't impose any order on the elements. A **Binary Search Tree (BST)** is a special type of binary tree where for each node:
*   All nodes in its *left* subtree have values *less than* the node's value.
*   All nodes in its *right* subtree have values *greater than* the node's value.
BSTs are very common due to their efficiency in searching, so our implementation examples will focus on them.

### 2. Why Use Binary Trees?

Binary trees offer several advantages:

*   **Hierarchical Data Representation:** Perfect for representing data with natural parent-child relationships (e.g., file systems, organization charts, family trees).
*   **Efficient Searching:** Binary Search Trees (BSTs) allow for searching, insertion, and deletion in O(log n) average time complexity (if reasonably balanced).
*   **Efficient Sorting:** An inorder traversal of a BST retrieves elements in sorted order.
*   **Expression Trees:** Used to represent mathematical expressions.
*   **Decision Trees:** Used in machine learning and decision analysis.

### 3. Representing Binary Trees in C

In C, we typically represent a tree node using a `struct`. Each node needs:

1.  **Data:** The value stored in the node (e.g., an integer, character, or pointer to more complex data).
2.  **Pointer to the Left Child:** A pointer of the same `struct` type.
3.  **Pointer to the Right Child:** Another pointer of the same `struct` type.

```c
#include <stdio.h>
#include <stdlib.h> // For malloc and NULL

// Define the structure for a tree node
struct Node {
    int data;           // The data stored in the node
    struct Node *left;  // Pointer to the left child
    struct Node *right; // Pointer to the right child
};

typedef struct Node Node; // Use 'Node' as a shorthand for 'struct Node'
```

We use pointers (`left` and `right`) because the children are also nodes of the same type, potentially located elsewhere in memory. If a node doesn't have a left or right child, the corresponding pointer will be `NULL`.

### 4. Common Operations with C Implementations

Let's implement some fundamental operations. We'll focus on a Binary Search Tree (BST) for insertion and search, as it's a practical and common use case.

#### a) Creating a New Node

We need a helper function to allocate memory for a new node and initialize its values.

```c
// Function to create a new node
Node* createNode(int data) {
    // Allocate memory for the new node
    Node* newNode = (Node*)malloc(sizeof(Node));

    // Check if memory allocation was successful
    if (newNode == NULL) {
        printf("Error: Memory allocation failed!\n");
        exit(1); // Exit if memory cannot be allocated
    }

    // Assign data to the node
    newNode->data = data;

    // Initialize left and right children to NULL
    newNode->left = NULL;
    newNode->right = NULL;

    return newNode; // Return the pointer to the newly created node
}
```

#### b) Insertion (BST Implementation)

For a BST, new nodes are inserted based on their value compared to existing nodes.

*   If the tree is empty, the new node becomes the root.
*   If the value is less than the current node's data, try inserting into the left subtree.
*   If the value is greater than or equal to the current node's data, try inserting into the right subtree.
*   Repeat recursively until an empty spot (`NULL` pointer) is found.

```c
// Function to insert a node into a BST (recursive)
Node* insertNode(Node* root, int data) {
    // 1. Base Case: If the tree/subtree is empty, create the node here
    if (root == NULL) {
        return createNode(data);
    }

    // 2. Recursive Step: Compare data and decide left/right
    if (data < root->data) {
        // If data is smaller, go to the left subtree
        root->left = insertNode(root->left, data);
    } else { // data >= root->data
        // If data is larger or equal, go to the right subtree
        // (Note: Handling duplicates - here we put them in the right subtree)
        root->right = insertNode(root->right, data);
    }

    // Return the (potentially updated) root pointer of this subtree
    return root;
}
```

**How it works:** The function returns the pointer to the node it was called with (or the new node if it was `NULL`). The crucial part is `root->left = insertNode(...)` and `root->right = insertNode(...)`. This ensures that the `left` or `right` pointer of the *parent* node gets correctly updated to point to the newly created node when the base case is reached.

#### c) Tree Traversal

Traversal means visiting each node in the tree exactly once. There are three common depth-first traversal methods:

1.  **Inorder (Left - Node - Right):** Visits the left subtree, then the current node, then the right subtree. For a BST, this yields elements in sorted order.
2.  **Preorder (Node - Left - Right):** Visits the current node, then the left subtree, then the right subtree. Useful for copying a tree or creating prefix expressions.
3.  **Postorder (Left - Right - Node):** Visits the left subtree, then the right subtree, then the current node. Useful for deleting a tree (as you delete children before the parent) or creating postfix expressions.

```c
// Inorder Traversal (LNR)
void inorderTraversal(Node* root) {
    // Base case: If the node is NULL, do nothing
    if (root == NULL) {
        return;
    }
    // 1. Traverse left subtree
    inorderTraversal(root->left);
    // 2. Visit the current node (print its data)
    printf("%d ", root->data);
    // 3. Traverse right subtree
    inorderTraversal(root->right);
}

// Preorder Traversal (NLR)
void preorderTraversal(Node* root) {
    if (root == NULL) {
        return;
    }
    // 1. Visit the current node
    printf("%d ", root->data);
    // 2. Traverse left subtree
    preorderTraversal(root->left);
    // 3. Traverse right subtree
    preorderTraversal(root->right);
}

// Postorder Traversal (LRN)
void postorderTraversal(Node* root) {
    if (root == NULL) {
        return;
    }
    // 1. Traverse left subtree
    postorderTraversal(root->left);
    // 2. Traverse right subtree
    postorderTraversal(root->right);
    // 3. Visit the current node
    printf("%d ", root->data);
}
```

#### d) Searching (BST Implementation)

Searching in a BST leverages the ordering property to quickly find a value.

*   Start at the root.
*   If the search value matches the current node's data, return the node.
*   If the search value is less than the current node's data, search in the left subtree.
*   If the search value is greater than the current node's data, search in the right subtree.
*   If you reach a `NULL` pointer, the value is not in the tree.

```c
// Function to search for a node in a BST (recursive)
Node* searchNode(Node* root, int key) {
    // 1. Base Cases:
    //   a) Root is NULL (subtree empty) or key is found at root
    if (root == NULL || root->data == key) {
        return root;
    }

    // 2. Recursive Step: Key is greater than root's data -> search right
    if (key > root->data) {
        return searchNode(root->right, key);
    }

    // 3. Recursive Step: Key is smaller than root's data -> search left
    return searchNode(root->left, key);
}
```

### 5. Putting It All Together: Sample `main` Function

Let's create a simple BST, insert some elements, traverse it, and search for a value.

```c
#include <stdio.h>
#include <stdlib.h>

// --- (Include the Node struct definition and the functions: ---
// --- createNode, insertNode, inorderTraversal,         ---
// --- preorderTraversal, postorderTraversal, searchNode  ---
// --- from the sections above here)                     ---

// Define the structure for a tree node
struct Node {
    int data;
    struct Node *left;
    struct Node *right;
};
typedef struct Node Node;

// Function to create a new node
Node* createNode(int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    if (newNode == NULL) {
        printf("Error: Memory allocation failed!\n");
        exit(1);
    }
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Function to insert a node into a BST (recursive)
Node* insertNode(Node* root, int data) {
    if (root == NULL) {
        return createNode(data);
    }
    if (data < root->data) {
        root->left = insertNode(root->left, data);
    } else {
        root->right = insertNode(root->right, data);
    }
    return root;
}

// Inorder Traversal (LNR)
void inorderTraversal(Node* root) {
    if (root == NULL) return;
    inorderTraversal(root->left);
    printf("%d ", root->data);
    inorderTraversal(root->right);
}

// Preorder Traversal (NLR)
void preorderTraversal(Node* root) {
    if (root == NULL) return;
    printf("%d ", root->data);
    preorderTraversal(root->left);
    preorderTraversal(root->right);
}

// Postorder Traversal (LRN)
void postorderTraversal(Node* root) {
    if (root == NULL) return;
    postorderTraversal(root->left);
    postorderTraversal(root->right);
    printf("%d ", root->data);
}

// Function to search for a node in a BST (recursive)
Node* searchNode(Node* root, int key) {
    if (root == NULL || root->data == key) {
        return root;
    }
    if (key > root->data) {
        return searchNode(root->right, key);
    }
    return searchNode(root->left, key);
}

// --- Main function to demonstrate BST operations ---
int main() {
    Node* root = NULL; // Start with an empty tree

    // Insert nodes into the BST
    root = insertNode(root, 50); // Root node
    insertNode(root, 30);
    insertNode(root, 20);
    insertNode(root, 40);
    insertNode(root, 70);
    insertNode(root, 60);
    insertNode(root, 80);

    /* The constructed BST looks like this:
              50
           /     \
          30      70
         /  \    /  \
       20   40  60   80
    */

    printf("Inorder traversal (sorted order): ");
    inorderTraversal(root);
    printf("\n");

    printf("Preorder traversal: ");
    preorderTraversal(root);
    printf("\n");

    printf("Postorder traversal: ");
    postorderTraversal(root);
    printf("\n");

    // Search for nodes
    int keyToSearch = 40;
    Node* foundNode = searchNode(root, keyToSearch);
    if (foundNode != NULL) {
        printf("Node with key %d found.\n", keyToSearch);
    } else {
        printf("Node with key %d not found.\n", keyToSearch);
    }

    keyToSearch = 90;
    foundNode = searchNode(root, keyToSearch);
    if (foundNode != NULL) {
        printf("Node with key %d found.\n", keyToSearch);
    } else {
        printf("Node with key %d not found.\n", keyToSearch);
    }

    // Remember to free the allocated memory in a real application!
    // (A function for freeing the entire tree would typically use postorder traversal)
    // freeTree(root); // Example - function not implemented here

    return 0;
}
```

**Note on Memory Management:** The example above doesn't include a function to `free` the memory allocated for the tree nodes. In a real-world application, you *must* free this memory to avoid memory leaks. A common way to do this is using a postorder traversal, freeing the children before freeing the parent node.

### 6. Complexity Considerations (BST)

*   **Search, Insertion, Deletion:**
    *   **Average Case:** O(log n) - This happens when the tree is reasonably balanced (height is proportional to log n).
    *   **Worst Case:** O(n) - This happens when the tree becomes skewed (like a linked list), for example, if you insert elements in strictly increasing or decreasing order. The height becomes proportional to n.

*   **Traversal:** O(n) - All traversal methods visit each node exactly once.

Because the performance of BSTs degrades in the worst case, self-balancing binary search trees (like AVL trees or Red-Black trees) were developed. They perform automatic rotations during insertions and deletions to ensure the tree remains balanced, guaranteeing O(log n) performance for major operations.

---

### Conclusion

Binary trees, especially Binary Search Trees, are powerful tools in a programmer's arsenal. They provide an efficient way to handle ordered data and represent hierarchical relationships. Understanding their structure, terminology, and basic operations (like insertion, traversal, and search) using a language like C is a crucial step in mastering data structures and algorithms. While this post covered the basics, there's much more to explore, including deletion algorithms, balancing techniques, and different tree variations. Happy coding!

### References

1.  **Textbooks:**
    *   **"Introduction to Algorithms" (CLRS) by Cormen, Leiserson, Rivest, and Stein:** Often considered the bible of algorithms. It provides rigorous theoretical foundations for binary trees, binary search trees, traversals, and complexity analysis (Chapters 10 and 12 in the 3rd/4th editions typically cover basic data structures including trees).
    *   **"Data Structures and Algorithm Analysis in C" by Mark Allen Weiss:** A highly respected textbook that specifically uses C for implementations. It covers binary trees, BSTs, traversals, and related concepts in detail, aligning well with the post's implementation focus.
    *   **"The C Programming Language" (K&R) by Kernighan and Ritchie:** While not focused on data structures specifically, it's the definitive reference for the C language itself, essential for understanding pointers, structs, `malloc`, and `stdlib.h` used in the implementations.

2.  **Online Resources:**
    *   **GeeksforGeeks (geeksforgeeks.org):** Has extensive articles and tutorials on Binary Trees, Binary Search Trees, traversals (Inorder, Preorder, Postorder), and provides code examples in various languages, including C. (e.g., search for "Binary Tree Data Structure", "Binary Search Tree C", "Tree Traversal C").
    *   **Programiz (programiz.com):** Offers clear, beginner-friendly tutorials on data structures, including binary trees and BSTs, often with C implementations. (e.g., search for "Binary Tree C", "BST C Programiz").
    *   **TutorialsPoint (tutorialspoint.com):** Another resource providing tutorials on data structures and algorithms with C examples.
