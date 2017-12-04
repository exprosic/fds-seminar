# Review on "Regular Expression Equivalence via Derivatives"

### Overall Structure

### Minor Corrections

- `Deriv x A = {xs. x # xs ∈ A}`
  - Although the name `Deriv` emphasizes its correspondence with `deriv`, it may also cause confusion in contexts where an explicit type annotation of its parameter is not present.
- 4.1 Language Coinduction
  - Explicit type annotations might be helpful here, since otherwise readers have to do type inference in mind after locating a possible starting point.
  - Since this lemma is the foundation of all following sections, it deserves a bit more explanation in plain text. Otherwise, the reader (like myself) might ignore a few simple yet important facts. For example, `shows K0 = L0` 