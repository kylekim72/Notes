## Questions
1. What is the key difference between dependent type vs refinement type?
Is there a difference between them in terms of expressiveness? For example, is there a certain property that can be described in dependent type system, but cannot be described in refinement type system?

A: No. Anytime when you refines type, it is refinement type. However, when the predicate in your refinement type depends on the program value, we call that dependent type.

2. Ivy uses EPR + relational abstraction as their background theory, and Ravencheck too(+ partial function semantics). Are there any particular advantages of using EPR? For example, is it easy to write some properties(safety, liveness, etc.) of Paxos, Raft, or any other distributed protocols in EPR compared to other decidable fragments?

A: Because EPR is the most expressive fragment. There are some other decidable fragments(GK?), but EPR allows infinite quantifications(in exists forall order, even in forall exists order if there is no sort cycle). It is easy to write same property in full FOL or second order logic, but it goes to undecidable.

## Mini lecture on F*
Assignment: Understand how the type checking in F*(read Ranjit's pldi 2008 2009 paper, only type checking parts) works, and try same thing in Ravencheck and compare pros and cons of these. In addition, also see the DARE2024.

```
module Welcome

val length: #a:Type -> list a -> nat
let rec length l = match l with
    | Nil -> 0
    | x::xs -> 1 + length xs

(*val concat : #(a:Type) -> l1:list a -> l2:list a -> l:list a{length l == length l1 + length l2}*)
let rec concat l1 l2 = match l1 with
    | Nil -> l2
    | x::xs -> x::(concat  xs l2)
    
val concat_preserves_length: #(a:Type) -> l1:list a -> l2:list a -> l:list a ->
        Lemma(requires l == concat l1 l2)
             (ensures length l == length l1 + length l2)
(* expanded version: 
  val concat_preserves_length: #(a:Type) -> l1:list a -> l2:list a -> l:list a 
            -> #(v1:(){l = concat l1 l2}) -> v2:(){length l = length l1 + length l2}
  let concat_preserves_length l1 l2 l = ()
   
   
*)
let rec concat_preserves_length l1 l2 l = match l1 with
    | Nil -> ()
    | x::xs -> concat_preserves_length xs l2 (concat xs l2)
```

From here, I will explain the code above.

```
val length: #a:Type -> list a -> nat
```
Define the type signature of length function. It takes a list that contains elements of type a, returns nat.
```
let rec length l = match l with
    | Nil -> 0
    | x::xs -> 1 + length xs
```
The body(implementation) of length function. Matches l and if l is `Nil`, then return 0. If l is `Cons x xs`, then return 1 + xs.

How does F* typechecks the legnth function? In this case, how does F* gurantees the termination of legnth function?
To prove the termination of the length function, F* considers the type of length as `#a:Type -> m:list a{ m << l } -> nat.` Here, the condition `m << l` acts as a measure indicating that the size of the argument strictly decreases with each recursive call. In our case, `xs << l` since l = Cons x xs, F* can prove the termination of length.


```
val concat : #(a:Type) -> l1:list a -> l2:list a -> l:list a{length l == length l1 + length l2}
```
Define the type signature of `concat` function. It refines list l with `length l == length l1 + length l2`.
```
let rec concat l1 l2 = match l1 with
    | Nil -> l2
    | x::xs -> x::(concat  xs l2)
```
Body of the `concat` function. First, F* checks the termination of concat function same as above. Since we know `xs << l1`, F* gurantees the termination of concat. Next, F* checks whether the return value of the match statement satisfies the refinement type.

1. Base case(l1 == Nil)
In this case, match statement returns l2 and F* try to check `length l2 == length Nil + length l2`. Since length Nil == 0, we conclude this is true.

2. Inductive case(l1 == x::xs)
In this case. match statement returns `x::(concat  xs l2)`. Since we've checked that `concat` is guranteed to terminate, we can describe `(concat  xs l2)` with the refinement type. Let `l'` be the return value of `(concat  xs l2)`, then `length l' == length xs + length l2` and this is our Inductive Hypothesis(IH).

In this branch, the return value of `concat` is `x::(concat  xs l2)` == `x::l'`. Thus, we need to prove `length(x::l') == length(x::xs) + length l2`. For LHS, `length(x::l')` can be reduced to `1 + length l'` by the definition of length. We know `length l' == length xs + length l2`, so LHS would be `1 + length xs + length l2`. For RHS, `length(x::xs)` can be reduced to `1 + length xs` by the definition of length. Now F* can conclude that LHS == RHS. We call this way of proof as intrinsic.



