# DP & Graph traversal

## Dynamic programming

> Dynamic programming is when we have a problem that we don't know how to solve, and we split it into smaller problems that we also don't know how to solve.

*Dynamic programming* is an approach that is somewhat similar to mathematical induction that usually boils down to the following steps:

* We find how the solution for a given set of parameters (e.g. $n$) is related to the solution for a smaller set of parameters;
* We create an array (or any other DS) to store answers for our subtasks;
* We initialize it with solutions for known small (i.e. base) cases;
* We traverse that structure from known to unknown parameter sets, updating optimal answers;
* We return the answer (it is likely already computed, or we descend back through this structure and rebuild the path to it)

It is helpful to give answers to following questions when solving a DP problem:

1. What do we store?
2. What are the base cases?
3. How to traverse the array? (or any other structure)
4. How to compute new values?
5. Where is the answer stored?

### Simple DP

> A grasshopper can jump either a single cell, or two cells forward. How many ways are there to get from the first cell to the $n$-th?

Let's follow the steps:

1. We store the solution for the $k$-th cell;
2. There is a single way to reach the first cell (i.e. $dp[0] \coloneqq 1$)
3. We iterate over the array indices, starting with $0$ up to $n$;
4. $dp[k] = dp[k - 1] + dp[k - 2]$;
5. The answer is in the $n$-th cell, that is $dp[n]$.

Here is a simple implementation:
```cpp title="Naive implementation"

uint64_t Solution(size_t n) {
  std::vector<uint64_t> ways_count(n); // zero-based indexing :)
  ways_count[0] = 1;

  for (size_t index = 1; index < n; ++index) {
    ways_count[index] = ways_count[index - 1] +
      (index > 1 ? ways_count[index - 2] : 0);
  }

  return ways_count[n - 1];
}
```

Note that we have to use only the last two values, so we can optimize our solution a little bit:

```cpp title="O(1) extra space implementation"

uint64_t Solution(size_t n) {
  uint64_t ways_count = 1;
  uint64_t previous_ways_count = 0;

  for (size_t index = 1; index < n; ++index) {
    std::tie(ways_count, previous_ways_count) =
      std::make_tuple(ways_count + previous_ways_count, ways_count);
  }

  return ways_count;
}
```

The same approach can be used in multidimensional cases or in case of subtrees, we just change the dimensions of the $dp$ array.

### Matrix DP

Actually, we can do faster than $O(n)$. We have a *linear recurrence relation with constant coefficients*: $dp_n = dp_{n - 1} + dp_{n - 2}$.

!!! note "Linear recurrence relations and matrices"
    If we have a following relation: $a_n = c_{n - 1} a_{n - 1} + c_{n - 2} a_{n - 2} + \ldots + c_1 a_1$, then we can add the following equations: $a_{n - 1} = a_{n - 1}, \ldots, a_2 = a_2$, and get a following matrix equation:
    $$
    \begin{pmatrix}
    a_{n} \\\\ a_{n - 1} \\\\ a_{n - 2} \\\\ \vdots \\\\ a_2
    \end{pmatrix} =
    \underset{A}{\underbrace{\begin{pmatrix}
      c_{n - 1} & c_{n - 2} & \ldots & c_1 \\\\
      1 & 0 & \ldots & 0 \\\\
      0 & 1 & \ldots & 0 \\\\
      \vdots & \ddots & \vdots & \vdots \\\\
      0 & \ldots & 1 & 0
    \end{pmatrix}}}
    \begin{pmatrix}
    a_{n - 1} \\\\ a_{n - 2} \\\\ a_{n - 3} \\\\ \vdots \\\\ a_1
    \end{pmatrix}.
    $$

    Observe that each multiplication by $A$ shifts the vector by one position. Now it's trivial to see that $a_{n + k}$ will be the first element of our vector, multiplied by $A^{k - 1}$ .

So the answer can be computed as such: $\begin{pmatrix} a_{n + 1} \\\\ a_{n} \end{pmatrix} = \begin{pmatrix} 1 & 1 \\\\ 1 & 0 \end{pmatrix}^n \cdot \begin{pmatrix} 1 \\\\ 0 \end{pmatrix}$.

We can calculate $A^n$ in $O(\log n)$ time due to following relation: $ A^n = \begin{cases} A^{n/2} \cdot A^{n/2} & 2 \mid n \\\\ A \cdot A^{n - 1} & 2 \nmid n \end{cases}$

```cpp title="Pseudocode"
// class Matrix { ... };
// class Vector { ... };

uint64_t Solution(size_t n) {
  Matrix multiplier = {{1, 1}, {1, 0}};
  Vector base_vector = {1, 0};

  while (n > 0) {
    if (n % 2 == 0) {
      multiplier *= multiplier;
      n /= 2;
    } else {
      multiplier *= Matrix{{1, 1}, {1, 0}};
      --n;
    }
  }

  return (multiplier * base_vector)[1];
}
```

### Bitmask DP

What if we need to depend on the answer on various subsets? The answer is to use a bitmask as one of the parameters, which will represent used elements.

* To check if a $k$-th element is used: ``` (mask & (1ULL << k)) != 0 ```;
* To add a $k$-th element into a subset: ``` mask |= (1ULL << k) ```;
* To remove a $k$-th element into a subset: ``` mask = mask &= ~(1ULL << k) ```;
* To "flip" a $k$-th element: ``` mask ^= (1ULL << k) ```;

### Different other optimizations

* [Knuth's optimization & CHT](https://usaco.guide/adv/dp-more?lang=cpp)
* [Profile DP](https://cp-algorithms.com/dynamic_programming/profile-dynamics.html)
* [Sum-over-subsets DP](https://usaco.guide/adv/dp-sos?lang=cpp)

## Graph traversal algorithms

TODO, but the structure is as such:

1. White-path lemma
2. DFS
3. BFS and why is it needed (same-distance nodes)
4. Cycle search
5. Topological sort
6. 0-k BFS
