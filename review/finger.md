## Review on Finger Trees

### Overall Structure

- The first paragraph of "1.1 Implicit Recursive Slowdown" seems using the not so well-known term "implicit recursive slowdown" for granted, which may give the reader a wrong impression before reading further, that it is a required prior knowledge rather than a topic to be introduced. I myself was a bit frightened by the term " recursive slowdown design principle", "refinement", "restrictions", "the originals" etc., which I had no idea what they are about before finishing reading the whole paper. Maybe put it in a separated "Related Works" section, which is supposed to contain such terms without explanation, or move it to the end of the section as a sub-conclusion.
- The applications introduced in section 3 are actually pretty helpful for motivating the reader to go through all the instantiations and manipulations in previous sections. Maybe give a brief description of these applications, without resorting to source code, in the beginning of the paper before dive into the details.
- Even written in an extremely concise and self-explanatory language, the Haskell code still presents a large amount of detail when being used to describe a non-trivial data structure. It is indeed very convincing as an argument, but less friendly as an introduction. A simple or even hand-written figure would be appreciated here.

### Minor Corrections

- all digits are ~~after using them~~ **used** as a weight to two to the power of their position
- Both inc and dec are implemented **in** the obvious way. 
- ...  we have lost our ~~save~~ **safe** digit. 
  - Why "lost"?
- Instead of having the ability ~~to~~ **of** representing a number **to** be the primary ~~function~~ **application** of this data structure, <u>it</u> **(?)** merely indicates the number of elements stored. 
  - "function" seems ambiguous here
- To enable fast access to the end of the data structure, each Deep node gets a second finger.
  - Why and what exactly does duplicating `Digit` help? Does "fast access" mean  an improved complexity or factor?
- without the need for dedicated commands ~~and required~~ for finger trees. 
- ... to provide a way ~~the~~ **to** summarize a collection of type a. 
- ... which is ~~required to of the~~ **an** instance of the Monoid class.
- `Node2 (Node3 1 2 3) (Node2 4 5) `
  - Isn't there a mismatch of arity with its definition `data Node v a = Node2 v a a | Node3 v a a a`?
- ~~The~~ after refining the original binary numbers into dequeues
- **(not only?)** Because they are important when defining these functions, but also for its intrinsic value,
- `node3 a b c = Node3 (µ a .+| b .+| c) a b c`
  - `(.+|)` has not been defined or explained in the paper
- In this case, ~~the~~ **it** implicitly calculate**s** the cached measurement.
- ... so we first check ~~of~~ that
- In order to do that, we **define** the `Split f a` type

