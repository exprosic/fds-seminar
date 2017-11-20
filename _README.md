# Zippers and the Algebra of Algebraic Datatypes

[TOC]

## Abstraction

Zipper is a data structure for 'modifying' the immutable variables of algebraic datatype like List and Tree, which are basic components to most functional programming languages. It models 'modification' of some part as the composition of its context and its replacement. Zipper, or context of algebraic datatype in general, is surprisingly similar with derivatives in calculus. This similarity can be partially explained in the context of combinatorial species.

## Zipper

Zipper is a data structure that helps manipulating functional datatype. Although functional programming paradigm doesn't allow a value to be in-place modified as in an imperative language, it is still useful to implement a function that combines a series of operations with a value, as illustrated in following sections.

### Ordered Tree

```isabelle
datatype 'a tree =
  Leaf
| Node (root: 'a) (subtrees: "'a tree list")
```
Imagine a pointer that initially points to the root of a tree. The operations being applied on this tree includes:

1. moving the pointer down to its first child, up to its parent, left to its previous sibling, or right to its next sibling;
2. replace the pointed tree with a new tree. 

A trivial representation of this pointer is a list of labels representing its path from the root, like `["Down", "Right", "Right", "Down"]`. But it would require linear time wrt. path length both to access and to replace a subtree. Instead, zipper models the pointer as a complete context of the pointed subtree:

```isabelle
datatype 'a tree_zipper =
  Teeth (siblings_left:  "'a tree list")
        (parent_root:    'a)
        (siblings_right: "'a tree list")
        (parent_zipper:  "'a tree_zipper")
| AlreadyTop
```

By defining a reconstruction function:

```isabelle
fun reconstruct_tree :: "'a tree_zipper => 'a tree => 'a tree" where
  "reconstruct_tree AlreadyTop t = t"
| "reconstruct_tree (Teeth lefts root rights parent_zipper) t =
    let current_subtree = (Node root (rev lefts @ [t] @ rights)) in
      reconstruct_tree parent_zipper current_subtree"
```

we can verify that our `tree_zipper` is indeed a complete context. Notice that the process of reconstructing a tree mimics a zipper being fastened bottom-up. `reconstruct_tree` need not be called at each modification. Instead, we can maintain a pair

 ```isabelle
type_synonym 'a tree_focus = 'a tree_zipper * 'a tree
 ```

which is effectively a "functional pointer". By changing its second component, we effectively replace the pointed subtree. `reconstruct_tree` can be called in the end if that is necessary at all. Operations upon the pointer can be defined as:

```isabelle
fun focus_down :: "'a tree_focus => 'a tree_focus" where
  "focus_down (current_zipper, Node root (first_subtree # rest_subtrees)) =
  	let zipper_down = (Teeth [] root rest_subtrees current_zipper) in
      (zipper_down, first_subtree)"

fun focus_up :: "'a tree_focus => 'a tree_focus" where
  "focus_up (Teeth lefts root rights parent_zipper, t) =
  	let reconstructed_parent = Node root (rev lefts @ [t] @ rights) in
      (parent_zipper, reconstructed_parent)"
    
fun focus_right :: "'a tree_focus => 'a tree_focus" where
  "focus_right (Teeth lefts root (first_right # rest_rights) parent_zipper, t) =
  	let zipper_right = (Teeth (first_right # lefts) root rights parent_zipper) in
      (zipper_right, t)"

fun focus_left :: "'a tree_focus => 'a tree_focus" where
  "focus_left (Teeth (last_left # rest_lefts) root rights parent_zipper, t) =
    let zipper_left = (Teeth rest_lefts root (last_left # rights) parent_zipper) in
      (zipper_left, t)"
      
fun replace_subtree :: "'a tree_focus => 'a tree => 'a tree_focus" where
  "replace_subtree (zipper, _) t = (zipper, t)"
```

For simplicity, impossible operations (like `focus_down` of Leaf) are left undefined here.

### Zipper for List

Zipper, interpreted as a datatype of context in general, can also be defined for List. The context of a sub-list (sub-term of the list, aka. suffix) is its preceding prefix, which is reversed here to accelerate the operations:

```isabelle
type_synonym 'a list_context = 'a list  (* represents a reversed prefix *)
type_synonym 'a list_focus = 'a list_context * 'a list
```

The operations on `list_focus` includes moving it to one step left or right (suppose the list is unrolled from left to right), and replace the focused sub-list:

```isabelle
fun move_right :: "'a list_focus => 'a list_focus" where
  "move_right (rev_prefix, x#xs) = (x#rev_prefix, xs)"
  
fun move_left :: "'a list_focus => 'a list_focus" where
  "move_left (x#rev_prefix, xs) = (rev_prefix, x#xs)"
  
fun replace_sublist :: "'a list_focus => 'a list => 'a list_focus" where
  "replace_sublist (rev_prefix, _) l = (rev_prefix l)"
```

Reconstruction is a simple concatenation:

```
fun reconstruct_list :: "'a list_context => 'a list => 'a list" where
  "reconstruct_list rev_prefix l = rev (rev_prefix) @ l"
```

### Zipper for List with Focus on an Element

If we want to focus on a single element rather than a whole sub-list, the context would be both its reversed preceding prefix and its following suffix:

```
type_synonym 'a list_context2 = 'a list * 'a list (* reversed prefix and normal suffix)
type_synonym 'a list_focus2 = 'a list_context2 * 'a
```

The operations are defined similarly as before:

```isabelle
fun move_right :: "'a list_focus2 => 'a list_focus2" where
  "move_right ((rev_prefix, s0#suffix), a) = ((a#rev_prefix, suffix), s0)"

fun move_left :: "'a list_focus2 => 'a list_focus2" where
  "move_left ((p0#rev_prefix, suffix), a) = ((rev_prefix, a#suffix), p0)"

fun replace_element :: "'a list_focus2 => 'a => 'a list_focus2" where
  "replace_element (context, _) a = (context, a)"
```

Reconstruction is still simple concatenations:

```isabelle
reconstruct_list2 :: "'a list_context2 => 'a => 'a list" where
  "reconstruct_list2 (rev_prefix, suffix) a = rev (rev_prefix) @ [a] @ suffix"
```

## Context of Algebraic Datatypes

So far, our zipper (or context) datatypes are defined in a rather ad-hoc manner. It is natural to ask whether it is possible to derive such datatype from the original one, as `tree_zipper` from `tree` and `list_context` or `list_context2` from `list`. It turns out that the derivation of such context from a datatype is very similar with the process of taking a derivative from a real function. But before that, let me first illustrate the algebraic nature of the datatypes that we are dealing with.

### Algebraic Datatypes

Datatypes defined by `datatype` command, such as `tree` and `list`, are all syntax sugars for so-called "algebraic datatypes". They are "algebraic" because they are constructed with similar elements as in algebra terms, which are unit (aka. $1$), summation and product. As a small yet comprehensive example, `list` is defined as:

```isabelle
datatype 'a list =
  Nil
| Cons 'a "'a list"
```

Here `|` indicates a sum type of both sides. `Nil` and `Cons` are just insignificant case names of whatever follows them. Following `Nil` is an implicit `()::unit`, while following Cons is a product of `'a` and `'a list`. Therefore `datatype 'a list` can be transcribed as $list(a) = 1 + a * list(a)$. In spite of the loss of semantical interpretation due to the lack of case names, $list(a)$ is structurally the same as `datatype 'a list`: it is either of a $1$, or a pair of an element $a$ and another $list(a)$.

Algebraic datatype follows many common algebraic laws. A minimal example of distribution law would be $(1+1)*a = a + a$, which can be rephrased as: `type_synonym 'a lr2 = bool * 'a` is *isomorphic* with  `datatype 'a lr = L 'a | R 'a`.

Algebraic datatype can also be composed like $list(tree(a))$, which stands for `'a tree list` in Isabelle.

Two subtleties here:

- First, unlike the commonly used $0$ in algebra, the "zero type", i.e. a type that accepts no value, is rarely used in programming since there is no usual way to declare it, but during a paper experiment, it sometimes helps constructing intermediate results, as we will see in following sections. It behaves like zero in algebra, which disappears in a sum and absorbs all others in a product.
- Second, $list(x) = 1 + list(x)$ is strictly speaking not a definition, but it indicates a least fix-point of $f(x) = 1+f(x)$, which means the smallest type satisfying $x=f(x)$. The type of all finite lists and the type of all possibly infinite lists are both valid fix-points here, but the smallest one is the former. The latter, aka. the greatest fix-point, is also widely used in real-world functional programming as in Haskell, but at the moment let us focus on the least fix-point.

### Inductive Definition

Now we can define the context type inductively on sum, product, composition, least fix-point and basic types. Here I try not to formalize the derivation into esoteric formulas but to rather illustrate its meaning and correctness based on its interpretation as a datatype and to abbreviate the notation whenever unambiguous.  For those who prefer formalized arguments, please refer to McBride's original paper.

Let us note a context (or a *hole*) of type $a$ in type $X$ by $C[a](X)$. `tree_zipper`, `list_context` and `list_context2` would then be written as $C[tree](tree)$, $C[list](list)$ and $C[a](list)$.

- For a type $f + g$, the  hole must occur in either of them, which means the context type of sum is the sum of context types:

  ​	$$C[a](f+g) = C[a](f) + C[a](g)$$

- For a datatype $f * g$, the hole occur in one of them, while the other remains unchanged and need to be carried by the context as well. Therefore its context type should be

  ​	$$C[a](f*g) = C[a](f) * g + f * C[a](g)$$.

- For a datatype $f(g(a))$ such as $list(tree(a))$, $a$ occurs either in $g$ which is parameterized by $f$, or in $f$ directly. If it occurs in a $g$, it can be located by first finding this $g$ in $f$ and then finding the $a$ in this $g$; If it occurs directly in f, then we can treat $g$ as a constant. Put into formula, 

  ​	$$C[a](f(g(a))) = C[a](f(a;g)) + C[g](f(g;a)) * C[a](g)$$ 

- For a recursive datatype $g(a) = f(a, g(a))$ such as $list(a) = 1 + a * list(a)$, we can derive context types on both sides and treat the resulting equation as an implicitly defined least fix-point of the context type. But be careful when you try to derive the context of a recursive type in it self like $C[list](list)$:  apart from the possibility that the hole of a list occurs inside a list, it is possible that the current list is itself a hole, which gives an extra $1$. For example,

  ​	$$C[list](list) = 1 + C[list](1 + x * list)$$

  where the left-most $1$ is the extra $1$ that represents such possibility. This special case may be eliminated by a likely lengthy formalization, but for now let us take the quick and dirty way.

- For basic cases, we have

  ​	$$C[a](1) = 0$$

  ​	$$C[a](a) = 1$$

  where $0$ indicates impossibility and behaves as described in previous sections, and $1$ indicates the one and only possibility of the hole's position. Again, be careful when $a$ is a recursive type.

All these rules shares the same structure of the correspondence rules for derivative in calculus:

​	
$$
\begin{eqnarray}
(f+g)'(x) &=& f'(x) + g'(x) \\
(f*g)'(x) &=& f'(x)*g(x) + f(x)*g'(x) \\
\frac{\partial}{\partial x}(f(x, g(x))) &=& \frac{\partial}{\partial x}(f(x, g)) + \frac{\partial}{\partial g}(f(x, g)) * \frac{\partial}{\partial x}(g(x)) \\
1' &=& 0 \\
x' &=& 1
\end{eqnarray}
$$

For the following sections, I will borrow this elegant notation for derivative $\frac{\partial}{\partial a}X$ as a synonym to $C[a](X)$ whenever unambiguous.

### Examples

Let us try to derive $\frac{\partial}{\partial L}L$ where $L$ stands for list :
$$
\begin{eqnarray}
\frac{\partial}{\partial L}L &=& 1 +\frac{\partial}{\partial L}(1 + a * L)\\
    &=& 1 + \frac{\partial}{\partial L}1 + \frac{\partial}{\partial L}(a*L)
\end{eqnarray}
$$

$\frac{\partial}{\partial L}1$ here represents the possibility of a context of `list` inside a `Nil`, which is nonexistent. But for the sake of consistency, let us follow the aforementioned rule and note it with "zero type". It will vanish among the summation in the end.
$$
\begin{eqnarray}
\frac{\partial}{\partial L}L &=& 1 + 0 + \frac{\partial}{\partial L}(a*L) \\
              &=& 1 + 0 + (\frac{\partial}{\partial L}a * L + a * \frac{\partial}{\partial L}L)
\end{eqnarray}
$$
again, `list` never occur in its parameter, thus $a$ is treated here as a constant and therefore $\frac{\partial}{\partial L}a$ is zero type.
$$
\begin{eqnarray}
\frac{\partial}{\partial L}L &=& 1 + 0 + (0 * L+ a * \frac{\partial}{\partial L}L) \\
&=& 1 + a * \frac{\partial}{\partial L}L
\end{eqnarray}
$$
which, as an implicitly defined least fix-point, is equivalent to $L = 1 + a * L$. That means $\frac{\partial}{\partial L}L = L$, which is exactly what we had before: `'a list_context = 'a list`.

For $\frac{\partial}{\partial a}L$, we have
$$
\begin{eqnarray}
\frac{\partial}{\partial a}L &=& \frac{\partial}{\partial a}(1 + a * L)\\
           &=& \frac{\partial}{\partial a}1+ \frac{\partial}{\partial a}(a*L) \\
           &=& 0       + (\frac{\partial}{\partial a}a* L + a * \frac{\partial}{\partial a}L) \\
           &=& 0       + (1       * L + a *\frac{\partial}{\partial a}L) \\
           &=& L + a *\frac{\partial}{\partial a}L
\end{eqnarray}
$$
Intuitively, this defines a type $a*a*\dots*a*L$, which is equivalent to $L*L$. This result, $\frac{\partial}{\partial a}L = L * L$, again coincide with the datatype we have defined before, namely `'a list_context2 = 'a list * 'a list'`.

As a final example, let us examine $\frac{\partial}{\partial a}T$ where $T$ stands for tree:

$$
\begin{eqnarray}
\frac{\partial}{\partial T}T &=& 1 + \frac{\partial}{\partial T}(1 + a * L(T)) \\
              &=& 1 + \frac{\partial}{\partial T}1 + \frac{\partial}{\partial T}(a * L(T)) \\
              &=& 1 + 0          + \frac{\partial}{\partial T}(a * L(T)) \\
              &=& 1 + 0          + (\frac{\partial}{\partial T}a * L(T) + a * \frac{\partial}{\partial T}L(T)) \\
              &=& 1 + 0          + (0          * L(T) + a * (L(T) * L(T)) * \frac{\partial}{\partial T}T) \\
              &=& 1 + a * L(T) * L(T) * \frac{\partial}{\partial T}L(T)
\end{eqnarray}
$$
This is indeed the same as `'a tree_zipper`.

### From Derivative back to Zipper

The `tree_zipper` for manipulating a Tree is bottom-up, but our derivation of derivative has a top-down semantic. Although sometimes equivalent, they are not meant to be used in the same way. For example, the translation of $\frac{\partial}{\partial a}T =  1 + a * L(T) * L(T) * \frac{\partial}{\partial T}L(T)$ would be

```isabelle
datatype 'a tree_deriv =
  Inside (root: 'a) (left: 'a tree list) (right: 'a tree list) (sub: 'a tree_deriv)
| Here
```

Its semantical difference from `'a tree_zipper` can be illustrated through its reconstruct function, which is no longer tail-recursive:

```
fun reconstruct :: "'a tree_deriv => 'a tree => 'a tree" where
  "reconstruct (Inside root left right sub)" = Node root (left @ [reconstruct sub t] @ right)
| "reconstruct Here t = t"
```

and moving such `tree_deriv` would require going through the whole path:

```
fun move_down :: "('a tree_deriv * 'a tree) => ('a tree_deriv * 'a tree)" where
  "move_down (Top, Node v (c0#children)) =
    (Inside v [] children Top, c0)"
| "move_down (Inside root left right inside, t) = 
    let (inside2, t2) = (move_down inside t) in
      (Inside root left right inside2, t2)"
```

it remains a question whether it is possible to construct a derivative like `tree_zipper` for all datatypes where a "reversed path" helps moving around efficiently. The answer would be yes, since the derivative of a least fix-points always contains such a path. Suppose a least fix-point is defined by $g(x) = f(x, g(x))$, then it's derivative is:
$$
\begin{eqnarray}
\frac{\partial}{\partial x}(g(x)) &=& \frac{\partial}{\partial x}(f(x, g(x))) \\
&=& \frac{\partial}{\partial x}(f(x, g)) + \frac{\partial}{\partial g}(f(x, g)) * \frac{\partial}{\partial x}(g(x)) \\
&\simeq& \frac{\partial}{\partial x}(f(x, g)) * L(\frac{\partial}{\partial g}(f(x, g)))
\end{eqnarray}
$$


The last step is similar with our final argument for $\frac{\partial}{\partial a}L$. Therefore, a list structure as the "route" in `tree_deriv` can always be found in any datatype derivatives, and as a list, it can always be reversed and interpreted bottom-up.

### A magical expression

By treating $L = 1 + a * L$ as an equation of numbers, we obtain

$$L = \frac{1}{1 - a}$$

This is nonsense as a type definition at first glance, but if we take derivative of both sides, we get $$\frac{\partial}{\partial a}L = \frac{1}{(1-a)^2} = L * L$$

which is exactly what we have obtained before. A nice explanation of such coincidence can be found in the theory of "combinatorial species".

## Combinatorial Species

### Definition

A **species** accepts a set of labels that fits into a certain structure. The species that are relevant here can be defined inductively as:

- The species $1$ accepts an empty set of labels. $2, 3, \dots$ are used as abbreviation for $(1+1), (1+1+1), \dots$ respectively, where $+$ is defined later.


- The species $X$ accepts a set with a single label.
- The product of two species $f(X)*g(X)$ accepts a set of labels by dividing them into two sets and let the two species accept them respectively. For example, the species $X*X$ defines a pair structure and accepts a label set $\{a, b\}$ in two ways: $(a, b)$ and $(b, a)$.
- The sum of two species $f(X) + g(X)$ accepts a set of labels by delegating the whole set to either of the two species. For example, $X + X*X$ accepts $\{a\}$ in one way by delegating it to $X$ and accepts $\{a, b\}$ in two ways by delegating it to $X*X$.
- The composition of two species $f(g(X))$ is a species $f(\cdot)$ defined in terms of $g(X)$. For example, if $f(s) = s * s$ and $g(X) = 1 + X$, then $f(g(x)) = (1+X)*(1+X)$ accepts $\{\}$ in one way, $\{a\}$ in two ways by giving the only label to either side, $\{a, b\}$ in one two ways by giving $a$ to the left and $b$ to the right or vice versa.
- The least fix-point defined by an equation of species is, as before, the smallest species satisfying such equation. For example the species $L$ defined by $L = 1 + X * L$  is equivalent to $1 + X + X^2 + X^3 + \dots$. It accepts $\{\}$ in one way, $\{a\}$ in one way, $\{a, b\}$ in 2 ways, $\{a,b,c\}$ in 6 ways, etc..

We can also define the derivative $f'(X)$ for a species $f(X)$. $f'(X)$ accepts a set of labels $\{a, b, \dots\}$ by delegating $\{*, a, b, \dots\}$ to $f(X)$ where $*$ is interpreted as a hole. For example, if $f(X)=X*X$, then $f'(X)$ accepts $\{a\}$ by delegating $\{*, a\}$ to $f(X)$ which results in two ways of acceptance: $(*,a)$ and $(a,*)$. For this specific example, it is apparent that $f'(X) = X + X$, where the right hand side also accepts $\{a\}$ in two ways. For an inductively defined "algebraic" species, one can show that the derivative of a species follows the same rule as in calculus. The idea is similar as when we define the context of algebraic datatypes. Note especially that $X^n = n * X^{n-1}$.

### Exponential Generating Function

One can prove inductively that all the inductively defined species is equivalent with a possibly infinite sum of powers of $X$. For each $X^n$, which accepts n values, there are $n!$ possibilities, which implies that the expression for a species can also be interpreted as an exponential generating function (EGF). Take list as an example, $f(X) = 1 + X + X^2 + \dots$ can be interpreted as $f(x) = 1 + 1!*\frac{x}{1!} + 2!*\frac{x^2}{2!} + ...$, which means that $f(X)$ has $1$ way of accepting empty set, $1!$ way for set of size $1$, $2!$ ways for set of size 2, etc.. Besides, operations (sum, product, etc.) on species results in isomorphic EGFs. Below are the cases for products and derivatives:

- Recall that a product of species $f(X)*g(X)$ accepts label sets of size $n$ by first dividing it into two sets of size $m$ and size $n-m$, then let $f(X)$ accepts those $m$ labels and $g(X)$ accepts the rest. If we note the number of possibilities by $f_m$ and $g_{n-m}$ respectively, then there are $\Sigma_{m=0}^{n}C(n,m)*f_m*g_{n-m}$ possibilities overall.  This is also the coefficient of term $x^n$ in the product of two EGFs $f(x)*g(x)$, which means that the EGF for species $f(X)*g(X)$ is $f(x)*g(x)$. 
- Recall that $(X^n)' = n * X^{n-1}$. Therefore, for a species with EGF $f(x) = 1 + f_1*\frac{x}{1!} + f_2*\frac{x^2}{2!} + ...$, its species derivative $f'(X)$ has EGF $f_1 + f_2*\frac{x}{1!} + f_3*\frac{x^2}{2!}...$, which is exactly $f'(x)$.

Let us now return to the species of list $L = 1 + X * L$. From the arguments above, we know that it stands both for the species and for its EGF. But as an equation of EGF, we are perfectly allowed to manipulate it into $L = \frac{1}{1-X}$. By taking derivative of $\frac{1}{1-X}$, we get the EGF for $L'$. Therefore, the magical equation can be interpreted in terms of EGF.

But is it possible to interpret just as a species? Yes. $\frac{1}{1-X}$ is *THE*  species that has EGF $\frac{1}{1-x}$. It is well defined because any two species with this egf have the same number of possibilities for accepting same label set, which indicates that these two species are isomorphic.

It is non-trivial to link species back to algebraic datatypes, but their structural similarities should be apparent.