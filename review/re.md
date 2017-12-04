# Review on "Regular Expression Equivalence via Derivatives"

### Overall Structure

- Formal definitions and proof scripts are indeed extremely convincing as argument, but might be less friendly as introduction. It might be helpful to instruct the reader to read the non-trivial ones if they were meant to be self-explanatory to some extent.
- Before diving into the detail of the loop body in section 5, it might be helpful to first describe the intent and the overall structure of the while-loop.

### Minor Corrections

- `Deriv x A = {xs. x # xs ∈ A}`
  - Although the name `Deriv` emphasizes its correspondence with `deriv`, it may also cause confusion in contexts where an explicit type annotation of its parameter is not present.
- 4.1 Language Coinduction
  - Explicit type annotations might be helpful here, since otherwise readers have to do type inference in mind after locating a possible starting point.
  - Since this lemma is the foundation of all following sections, it deserves a bit more explanation in plain text. Otherwise, the reader (like myself) might ignore a few simple yet important facts. For example, `shows K0 = L0` is directly related to the very equivalence that the paper is all about, and the lemma is a coinduction rule for such equivalence, instead of some auxiliary congruence rule.
- `nderiv` is a variant of `deriv`, explained below.
  - might better tell the reader that to what extent can they be treated the same, so that the validity of the arguments is not affected by this variance before it is explained pages below.
- ... given `as` and `ps`, it tests whether the REs in `ps` contain only atoms in `as` ...
  - Why does this matter for a certificate checker?
- However, it turns out we can iteratively construct `ps` using the same idea, such that the termination the while-loop already guarantees the premises of `bisim-lang-eq`.
  - Till this point, the reader has not yet known what **the** while-loop is and how it relates to `bisim-lang-eq`. Maybe just "the termination of **a** while-loop already guarantees the falsehood of its loop condition"
- The approach is to build the relation by adding a pair (**to `ps`**) that’s missing for the`is-bisimulation` property in every step.
- In each step, a pair is taken from <u>the work set</u>, added to <u>the relation</u>, and its derivatives are in turn added to the work set if they are not yet accounted for:
  - might better clarify the terms first
- ... However, the generated executable code only uses the unfolding equation ...
  - When and why did "generated executable code" come into play? Isn't everything inside the Isabelle/HOL framework?