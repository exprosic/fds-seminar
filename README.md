# Zippers and the Algebra of Algebraic Data Types

## Introduction

Zipper is a programming technique of manipulate functional datatype. Suppose, for example, we have a tree:
```isabelle
datatype 'a tree =
  Leaf
| Node 'a "'a tree list"
```
and we want to manipulate this tree with a sequence of operations: up, down, left, right
## Examples

### Binary Tree

```isabelle
datatype 'a binary_tree =
  Leaf
| Node 'a "'a binary_tree" "'a binary_tree"
```

```isabelle
datatype 'a binary_tree_context =
  Here
| InsideL 'a "'a binary_tree_context" "'a binary_tree"
| InsideR 'a "'a binary_tree" "'a binary_tree_context"
```

### Tree

```isabelle
datatype 'a tree =
  Leaf
| Node 'a "'a tree list"
```

```isabelle
datatype 'a tree_context =
  Here
| Inside 'a "'a tree list * 'a tree_context * 'a tree list"
```

```isabelle
datatype 'a tree_context' =
  Here "'a tree list"
| Inside 'a "'a tree list * 'a tree_context' * 'a tree list"
```

## Derivative

$$
\begin{eqnarray} 
\delta_X[1] &=& \emptyset \\
\delta_X[X] &=& 1 \\
\delta_X[U(X) + V(X)] &=& \delta_X[U(X)] + \delta_X[V(X)] \\
\delta_X[U(X) * V(X)] &=& \delta_X[U(X)] * V(X) + U(X) * \delta_X[V(X)] \\
\delta_X[U(V(X))] &=& \delta_V[U(V)](X) * \delta_X[V(X)] \\
\end{eqnarray}
$$

$$
\begin{eqnarray} 
\delta_X[\mu U(X)]] &=& \delta_X[U(\mu U(X), X)] \\
&=& \delta_V[U(V,X)] * \delta_X[\mu U(X)] + \delta_V[U(\mu U, V)] \\
&=&\mu W. \delta_V[U(V,X)] * W + \delta_V[U(\mu U, V)] \\
&=& list(\delta_V[U(V,X)]) * \delta_V[U(\mu U, V)]
\end{eqnarray}
$$

A derivation for $tree$:

$$
\begin{eqnarray}
\partial_X[tree(X)]
&=& \partial_X[1 + X * list(tree(X))] \\
&=& \partial_X[1] + \partial_X[X * list(tree(X))] \\
&=& \partial_X[1] + \partial_X[X] * list(tree(X)) + X * \partial_X[list(tree(X))] \\
&=& \partial_X[1] + \partial_X[X] * list(tree(X)) + X * \partial_{a}[list(a)]|_{a=tree(X)} * \partial_X[tree(X)] \\
&=& \partial_X[1] + \partial_X[X] * list(tree(X)) + X * list(a)^2|_{a=tree(X)} * \partial_X[tree(X)] \\
&=& 0 + 1 * list(tree(X)) + X * list(tree(X))^2 * * \partial_X[tree(X)] \\
&=& list(tree(X)) + X * list(tree(X))^2 * \partial_X[tree(X)]
\end{eqnarray}
$$
which is exactly the same as ``datatype tree_context'`` defined in previous sections.

$$
\begin{eqnarray}
\partial_X[L] &\triangleq& \partial_X[\mathrm{lfp}(U)(X)] \\
&=& \partial_X[U(L)(X)] \\
&=& \partial_{a}[U(a)(X)]|_{a=L} * \partial_X[L] + \partial_{b}[U(L)(b)]|_{b=X} * \partial_X[X] \\
&=& \partial_{a}[U(a)(X)]|_{a=L} * \partial_X[L] + \partial_{b}[U(L)(b)]|_{b=X} \\
&=& \mu V. (\partial_{a}[U(a)(X)]|_{a=L} * V + \partial_{b}[U(L)(b)]|_{b=X}) \\
&=& list(\partial_{a}[U(a)(X)]|_{a=L}) * \partial_{b}[U(L)(b)]|_{b=X}
\end{eqnarray}
$$



## Combinatorial Species
