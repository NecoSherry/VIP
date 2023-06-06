---
VIP: 
Title: The Ultimate State Pruner
Description: This proposal is based on VIP-211 and aims to achieve efficient state pruning by utilizing new nodes to mask old nodes.
Author: Some Author(author@foo.com)
Discussions: https://sherry.discourse.group/t/add-vip-the-ultimate-state-pruner/17
Category:  Core
Status: Draft
CreatedAt: 2023-06-05
  
---

# Overview

This proposal is based on [VIP-211](./VIP-211-zh_CN.md) and aims to achieve efficient state pruning by utilizing new nodes to mask old nodes.

  
# Rationale
## Basic Principle
 
To simplify the description, a binary tree is used as an example instead of the 16-ary tree in the Modified Patricia Tree (MPT).

Let's consider a state tree  *T<sub>N</sub>* with a commit number *N*. When traversing *T<sub>N</sub>*, we have nodes *a, b...g*, as shown below:

<pre>
T<sub>N</sub>
        a
       / \
      b   e
    / |   | \
   c  d   f  g
</pre>

  
After several updates to *T<sub>N</sub>*, we obtain *T<sub>N'<sup></sup></sub>*:

<pre>
T<sub>N<sup>'</sup></sub>
        a<sup>'</sup>
       / \
      b<sup>'</sup>  e
    / |   | \
   c<sup>'</sup> d<sup>'</sup>  f  g
</pre>
  
In *T<sub>N<sup>'</sup></sub>*, nodes *a, b, c, d* are updated to *a<sup>'</sup>, b<sup>'</sup>, c<sup>'</sup>, d<sup>'</sup>* respectively, and the old nodes *a, b, c, d* are masked by the new nodes. When the corresponding block *N<sup>'</sup>* of *T<sub>N<sup>'</sup></sub>* is highly likely to be irreversible, the old nodes can be safely deleted.


  
# Motivation
  
Aim to achieve efficient state pruning by utilizing new nodes to mask old nodes.
  

  

# Test Cases
  
<!--
  This section is optional for non-Core VIPs.

  The Test Cases section should include expected input/output pairs, but may include a succinct set of executable tests. It should not include project build files. No new requirements may be be introduced here (meaning an implementation following only the Specification section should pass all tests here.)
  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/vip-####/`. External links will not be allowed

  TODO: Remove this comment before submitting
-->
  
# Implementation
  
## Path Encoding
  
For the ease of implementation and performance considerations, an extension to the node storage format is needed based on VIP-211. The extended format is as follows:
  
    pack(path) || commitNum || hash(node) => ...
  
It means that the path information of the node is added before the node's storage key.

The pack(path) is denoted as *PP*, which is a 64-bit fixed-length encoding of the path. The encoding algorithm can be implemented in Go using the following code:

```go
func pack(path []byte) uint64 {
    n := len(path)
    if n > 15 {
        n = 15
    }
    var packed uint64
    for i := 0; i < 15; i++ {
        if i < n {
            packed |= uint64(path[i])
        }
        packed <<= 4
    }
    return packed | uint64(n)
}
```
  
It can be observed that the encoded result follows the dictionary order, which is consistent with the traversal order of the MPT. This significantly improves the speed of MPT traversal.
  
## Tracking New Nodes
  
Let *C* represent the commit number of a node. By traversing T<sub>N<sup>'</sup></sub> and applying the filtering condition *C* > *N*, along with the property stated in [VIP-211](./VIP-211-zh_CN.md):

> The commit number of a parent node is always greater than or equal to the commit number of its child nodes.

we can quickly find all the new nodes and build a mapping *m*. *m* is defined as:

    PP => C

which represents the mapping of the path of a new node to its commit number.
  
## Cleaning Old Nodes
  
Traverse the database to obtain the sequence of all nodes. Process the nodes in the following steps:

  * Extract the path *PP* and the commit number *C* from the current node's storage key. Let *C<sup>'</sup> = m[PP]*.

* If *C < C<sup>'</sup>*, it indicates that the current node is an old node that can be cleaned. Delete it directly.

* Otherwise, the current node is not masked by new nodes or it is a new node itself.
  
  
# Security Considerations

<!--
  All VIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. VIP submissions missing the "Security Considerations" section will be rejected. An VIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

# References
