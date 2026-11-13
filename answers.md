## 1. Shortest shortest paths

**1a)** The code is in `main.py`.

You don't need a new algorithm here. You just run Dijkstra and change what the heap sorts by. Instead of keying each entry on the path weight alone, you key it on the pair $(weight, edges)$ and compare pairs left to right. So the heap still pops the lightest path first. But when two paths weigh the same, the one with fewer edges comes out first and wins.

Why is that safe? Dijkstra only needs one thing to be true, which is that extending a path never makes it look better. With positive weights, adding an edge makes the weight go up, and it also makes the edge count go up. So the pair only ever grows. That's all the proof of Dijkstra relies on, so it still works with pairs.

Look at vertex $d$ in the test. You can reach it by $s \to c \to d$ for $4 + 3 = 7$, or by $s \to a \to b \to d$ for $1 + 2 + 4 = 7$. Same weight. But the first one has 2 edges and the second has 3, so $(7, 2)$ pops first and that's the answer you keep.


**1b)  ** Work and span 

It's the same cost as normal Dijkstra, because comparing two pairs still takes constant time.

Work is $O(|E| \log |E|)$, which is the same thing as $O(|E| \log |V|)$ since $|E| \le |V|^2$. Every edge can push one entry onto the heap, and each push or pop costs a log.

Span is also $O(|E| \log |E|)$. Dijkstra settles one vertex at a time and each step depends on the one before it, so there's nothing to run in parallel. That's the honest answer and it's what the book says about Dijkstra in Chapter 57.



## 2. Expression Trees


**2a)  ** Expression tree algorithm with work and span

Read the expression left to right and keep two stacks. One holds finished subtrees and the other holds operators.

When you see a literal, make it a leaf and push it onto the tree stack. When you see an operator or an opening parenthesis, push it onto the operator stack. When you hit a closing parenthesis, pop the operator, pop the top two subtrees, and make a new node with the operator on top and those two subtrees as its children. Then push that node back onto the tree stack. A unary minus like $(-3)$ is the same idea with one child. When the input runs out, the one tree left on the stack is $T_E$.

Each symbol gets pushed once and popped once, and every pop does constant work. So the work is $O(n)$.

The span is also $O(n)$ because you're scanning one symbol at a time and every step needs the stacks from the step before.

You can do better on span if you want to push it. Use a scan to compute the nesting depth of every symbol in $O(n)$ work and $O(\log n)$ span. Matching parentheses then become a parallel search problem, and the tree comes out with $O(n)$ work and $O(\log n)$ span. But the stack version is the one to know cold.


**2b)  ** Corresponding arithmetic expression work and span?

Evaluate bottom up. Leaves just look up their value. An internal node waits for both children and then applies its operator. And the two children don't depend on each other, so you can evaluate them in parallel.

Every node does constant work and there are $O(n)$ nodes, so the work is $O(n)$.

The span is the height of the tree, $O(h)$, because the longest chain of waiting goes from the deepest leaf up to the root. If the tree is balanced that's $O(\log n)$. If it's a long chain like $((((a+b)+c)+d)+e)$ then it's $O(n)$ and you get no speedup at all.


**2c)  ** work/span to update a previously computed value

Keep the value you computed at every internal node. Now when one variable changes, only the nodes above that leaf can change. Everything else in the tree is exactly what it was.

So you walk from the leaf up to the root and recompute each node using its new child and the stored value of its other child. That's constant work per node and you touch one node per level.

Work and span are both $O(h)$. That's $O(\log n)$ for a balanced tree and $O(n)$ in the worst case. It's a sequential walk up one path, so work and span come out the same.


## 3. Improving Dijkstra


**3a)  ** Maximum depth of a $d$-ary heap

Level $i$ holds up to $d^i$ nodes. So the number of levels you need for $n$ nodes grows like $\log_d n$.

The maximum depth is $\lceil \log_d n \rceil$, which is $O(\log_d n) = O\left(\frac{\log n}{\log d}\right)$.

Bigger $d$ means a flatter, wider tree. Keep that in mind because it matters in 3d.


**3b)  ** Work done by `delete-min` and `insert` operations in a $d$-ary heap.

These two are different and that's the whole point of the question.

Insert is $O(\log_d n)$. The new element starts at the bottom and swaps upward. At each level you only compare it with its one parent, so it's constant work per level.

Delete min is $O(d \log_d n)$. The element you moved to the root has to sink down. At each level you have to find the smallest of the $d$ children before you know where to swap, and that takes $d$ comparisons.

So making $d$ bigger helps insert and hurts delete min.


**3c)  ** New bound on the work for Dijkstra's using $d$-ary heap 

Count how often Dijkstra calls each operation. Inserts happen once per edge you relax, so there are $O(|E|)$ of them. Delete min happens once per vertex you settle, so $O(|V|)$ of them. The heap never holds more than $|E|$ things, and $\log_d |E| = O(\log_d |V|)$.

Put it together and the work is

$$O\big(|V| \cdot d \log_d |V| \;+\; |E| \log_d |V|\big)$$

The first part is the delete mins and the second part is the inserts. If you use the lazy version from the book that leaves stale entries in the heap, you can get up to $|E|$ delete mins instead. Just say which version you're analyzing.


**3d)  ** What value of $d$ yields an overall running time of $O(|E|)$?
 
Pick $d = |V|^{\epsilon}$. That's the same thing as $d = |E| / |V|$, the average degree.

Here's why it works. With that choice $\log_d |V| = \frac{\log |V|}{\epsilon \log |V|} = \frac{1}{\epsilon}$, which is a constant because $\epsilon$ is fixed.

The insert term becomes $|E| \cdot \frac{1}{\epsilon} = O(|E|)$.

The delete min term becomes $|V| \cdot |V|^{\epsilon} \cdot \frac{1}{\epsilon} = \frac{|E|}{\epsilon} = O(|E|)$.

Both are $O(|E|)$, so the whole thing is $O(|E|)$. You've basically made the tree so flat that its height is constant. And the extra cost of scanning $d$ children only happens $|V|$ times, which adds up to exactly $|E|$.


## 4. All-pairs shortest paths using Dynamic Programming

**4a)  ** Consider the following graph with 3 vertices.

![apsp_example.jpg](apsp_example.jpg)

Compute $\mathit{APSP}(i, j, k)$ for all $i, j, k$.

The edges are $\{0,1\}$ with weight $-2$, $\{0,2\}$ with weight $2$, and $\{1,2\}$ with weight $1$. The graph is undirected so every table is symmetric.

I'm reading "only vertices $0 \ldots k$ allowed" as the vertices you pass through in the middle. The endpoints $i$ and $j$ are always allowed. One more thing to flag. An undirected edge with a negative weight is technically a negative cycle, because you could bounce $0 \to 1 \to 0$ forever. So we count simple paths only, which is what the question clearly intends.

The table for $k = -1$ is the starting point with no middle vertices at all, just the direct edges.

| $k=-1$ | 0 | 1 | 2 |
|---|---|---|---|
| **0** | 0 | $-2$ | 2 |
| **1** | $-2$ | 0 | 1 |
| **2** | 2 | 1 | 0 |

Now let vertex 0 sit in the middle. The only pair that can use it is 1 and 2. Going $1 \to 0 \to 2$ costs $-2 + 2 = 0$, which beats the direct edge of 1.

| $k=0$ | 0 | 1 | 2 |
|---|---|---|---|
| **0** | 0 | $-2$ | 2 |
| **1** | $-2$ | 0 | **0** |
| **2** | 2 | **0** | 0 |

Now let vertex 1 sit in the middle too. Going $0 \to 1 \to 2$ costs $-2 + 1 = -1$, which beats the direct edge of 2.

| $k=1$ | 0 | 1 | 2 |
|---|---|---|---|
| **0** | 0 | $-2$ | **$-1$** |
| **1** | $-2$ | 0 | 0 |
| **2** | **$-1$** | 0 | 0 |

Finally let vertex 2 in. The only new option is $0 \to 2 \to 1$ for $2 + 1 = 3$, and that's worse than $-2$. So nothing changes.

| $k=2$ | 0 | 1 | 2 |
|---|---|---|---|
| **0** | 0 | $-2$ | $-1$ |
| **1** | $-2$ | 0 | 0 |
| **2** | $-1$ | 0 | 0 |

The final answers are $\mathit{APSP}(i,j,2)$, the last table.


**4b)  ** Relationship between $\mathit{APSP}(i, j, 1)$ and
$\mathit{APSP}(i, j, 2)$. Can you write $\mathit{APSP}(i, j, 2)$ in
terms of $\mathit{APSP}(i, j, 0)$ and $\mathit{APSP}(i, j, 1)$ only?

Watch what happened every time we let a new vertex in. The best path either ignored the new vertex, or it went through it exactly once. If it goes through vertex 2, it splits into a piece from $i$ to 2 and a piece from 2 to $j$. Neither piece uses 2 in its middle, so both pieces are already in the $k = 1$ table.

So you only need the previous table.

$$\mathit{APSP}(i, j, 2) = \min\big(\mathit{APSP}(i, j, 1),\; \mathit{APSP}(i, 2, 1) + \mathit{APSP}(2, j, 1)\big)$$

And the $k = 1$ table came from the $k = 0$ table in exactly the same way, with 1 in place of 2. You never have to look further back than one step.


**4c)  ** Generalize your observation from b) above to give an optimal substructure property for $\mathit{APSP}(i, j, k).$

Swap the 2 for a general $k$.

$$\mathit{APSP}(i, j, k) = \min\big(\mathit{APSP}(i, j, k-1),\; \mathit{APSP}(i, k, k-1) + \mathit{APSP}(k, j, k-1)\big)$$

The base case is $\mathit{APSP}(i, j, -1) = w(i, j)$ if there's an edge, $0$ if $i = j$, and $\infty$ otherwise.

Here's the argument in words. Take the best path from $i$ to $j$ that only uses $0 \ldots k$ in the middle. Either it skips $k$, and then it's the best path using $0 \ldots k-1$. Or it uses $k$ once, and then both halves have to be the best paths using $0 \ldots k-1$. If either half weren't the best, you could swap in a better half and get a cheaper path overall. That would be a contradiction. This is Floyd Warshall.


**4d)  ** How many distinct subproblems will be computed from scratch, and
what is the resulting work of this dynamic programming algorithm?

There are $n$ choices for $i$, $n$ for $j$, and $n$ for $k$. So you get $n^3$ distinct subproblems.

Each one takes a min of two things and one addition. That's $O(1)$ work once its subproblems are already memoized.

The total work is $O(n^3)$.

If you also want span, you can fill in every $(i, j)$ for a fixed $k$ at the same time because they only read the $k-1$ table. So you do $n$ rounds of constant span and the span is $O(n)$.


**4e)  ** Compare the work of this algorithm against that of just running
Dijkstra $n$ times. Is our dynamic programming algorithm better than this?

Running Dijkstra from every vertex costs $n \cdot O(m \log n) = O(n m \log n)$, where $m = |E|$.

It depends on how many edges you have.

On a dense graph $m$ is close to $n^2$. Then Dijkstra $n$ times is $O(n^3 \log n)$ and the DP wins with $O(n^3)$.

On a sparse graph $m$ is around $n$. Then Dijkstra $n$ times is $O(n^2 \log n)$, which beats $O(n^3)$ by a lot.

But there's one more thing that matters. Dijkstra breaks with negative edge weights, and this DP doesn't care as long as there's no negative cycle. The graph in 4a has a negative edge, so Dijkstra wouldn't even give you the right answer there. For that case you'd compare against Johnson's algorithm from Chapter 59 instead.



## 5. Spanning trees

**5a)  ** Is a solution to MST guaranteed to be a solution to MMET? Why or why not?

Yes. Every MST is also a minimum maximum edge tree.

Suppose not. Let $T$ be an MST and let $e$ be its heaviest edge with weight $w$. Now say some other spanning tree $T'$ has every edge lighter than $w$.

Take $e$ out of $T$. That splits $T$ into two pieces, which is a cut of the graph. $T'$ is a spanning tree, so it has to cross that cut somewhere with some edge $e'$. And $w(e') < w$ because every edge in $T'$ is lighter than $w$.

Now put $e'$ into $T$ where $e$ used to be. You get a spanning tree that's lighter than $T$. But $T$ was supposed to be minimum. Contradiction.

The other direction is false though. An MMET doesn't have to be an MST. Think of a triangle with weights 1, 5 and 5. The MST is $\{1, 5\}$. But $\{5, 5\}$ also has max edge 5, so it's an MMET too, and its total weight is worse.


**5b)  ** Suppose that the optimal solution to MST is impossible to use for some reason. Describe an algorithm to instead find the next best tree (pseudo-code or English is fine). That is, return the tree with the next lowest weight. 

The key fact is that the second best tree is always just one swap away from the MST. You add one edge that isn't in the tree and take out one edge that is.

So here's the plan.

Build the MST $T$ with Kruskal or Prim.

For every edge $(u, v)$ that's not in $T$, think about adding it. That creates exactly one cycle, made of $(u, v)$ plus the path from $u$ to $v$ inside $T$. To get a tree back you have to remove an edge from that path. You want the result as light as possible, so remove the heaviest edge on the path. The new tree weighs $w(T) + w(u, v) - \mathit{maxOnPath}(u, v)$.

Try that for every non tree edge and keep the cheapest result. That's your second best tree.

```
T = MST(G)
best = infinity
for each edge (u, v) not in T
    m = heaviest edge on the path from u to v in T
    cost = weight(T) + w(u, v) - w(m)
    if cost < best
        best = cost
        answer = T with m removed and (u, v) added
return answer
```

The trick that makes this fast is finding the heaviest edge on a tree path quickly. Do a BFS or DFS from every vertex inside $T$ and record the heaviest edge seen on the way to each other vertex. Then every lookup in the loop is $O(1)$.


**5c)  ** What is the work of your algorithm?

The MST costs $O(m \log n)$.

The BFS from every vertex costs $O(n)$ each since $T$ has $n - 1$ edges. That's $O(n^2)$ total, and it fills a table with the heaviest edge for every pair.

The loop touches each of the $m$ edges once and does an $O(1)$ lookup. That's $O(m)$.

Add it up and the total work is $O(m \log n + n^2)$.

If you skip the table and walk the path fresh for every edge, each walk can cost $O(n)$. That gives $O(mn)$, which is still correct but slower on dense graphs.
