## Questions
1. What is the key difference between dependent type vs refinement type?
Is there a difference between them in terms of expressiveness? For example, is there a certain property that can be described in dependent type system, but cannot be described in refinement type system?

A: No. Anytime when you refines type, it is refinement type. However, when the predicate in your refinement type depends on the program value, we call that dependent type.

2. Ivy uses EPR + relational abstraction as their background theory, and Ravencheck too(+ partial function semantics). Are there any particular advantages of using EPR? For example, is it easy to write some properties(safety, liveness, etc.) of Paxos, Raft, or any other distributed protocols in EPR compared to other decidable fragments?

A: 

## Mini lecture on F*
Assignment: Understand how the type checking in F*(read Ranjit's pldi 2008 2009 paper, only type checking parts) works, and try same thing in Ravencheck and compare pros and cons of these. In addition, also see the DARE2024.

```
module Welcome

val length: #(a:Type) -> list a -> nat
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



