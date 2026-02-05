## Intro

In this note, I will talk about what Ravencheck outputs for counterexample currently(especially, for inductive verification such as TIP benchmarks), and how can we improve this into a more readable format. I will start with simple property that only requires 1 lemma to be proved.

## Problem description

First, below is the definition of Nat type and add function that is defined recursively.


```
enum Nat {
    // The zero variant
    Z,
    // The successor variant.
    S(Box<Nat>)
}
#[define]
#[recursive]
fn add(a: Nat, b: Nat) -> Nat {
    match a {
        Nat::Z => b,
        Nat::S(a_minus) => Nat::S(Box::new(add(*a_minus,b))),
    }
}
```

Given the recursive definitions above, I want to prove `add(S(a), b) == add(a, S(b))`, which can be expressed as below.

```
#[annotate]
#[inductive(a: Nat, b: Nat)]
fn add_move_s() -> bool {
    add(Nat::S(a), b) == add(a, Nat::S(b))
}
```

## Manual proof

In this section, I will write a manual proof for `add(S(a), b) == add(a, S(b))`. We need to check 4 cases, `a = Z and b = Z`(base case), `a = S(a_m) and b = Z`, `a = Z and b = S(b_m)`, and `a = S(a_m) and b = S(a_m)`. 

1. `a = Z and b = Z`
```
LHS = add(S(Z), Z)
add(S(Z), Z) = S(add(Z, Z)) (by unrolling)
S(add(Z, Z)) = S(Z) (by add's definition)

RHS = add(Z, S(Z))
add(Z, S(Z)) = S(Z) (by add's definition)

Done.
```
2. `a = S(a_m) and b = Z`
```
LHS = add(S(S(a_m)), Z)
add(S(S(a_m)), Z) = S(add(S(a_m), Z)) (by unrolling)

RHS = add(S(a_m), S(Z))
add(S(a_m), S(Z)) = S(add(a_m, S(Z))) (by unrolling)

We can apply IH here(a: a_m, b = Z)
IH: add(S(a), b) == add(a, S(b))

Done.
```
3. `a = Z and b = S(b_m)`
```
LHS = add(S(Z), S(b_m))
add(S(Z), S(b_m)) = S(add(Z, S(b_m))) (by unrolling)
S(add(Z, S(b_m))) = S(S(b_m)) (by add's definition)

RHS = add(Z, S(S(b_m)))
add(Z, S(S(b_m))) = S(S(b_m)) (by add's definition)

Done.
```

4. `a = S(a_m) and b = S(b_m)`

LHS = add(S(S(a_m)), S(b_m))
add(S(S(a_m)), S(b_m)) = S(add(S(a_m), S(b_m))) (by unrolling)

RHS = add(S(a_m), S(S(b_m)))
add(S(a_m), S(S(b_m))) = S(add(a_m, S(S(b_m)))) (by unrolling)

We can apply IH here(a: a_m, b: S(b_m) = b) since those are smaller substrcture of IH.
IH: add(S(a), b) == add(a, S(b))

Done.

## TIP Benchmarks

Table 1 reflects several updates to our TIP benchmarks case study. First, we discovered a soundness bug in our implementation of multi-function inductive verification conditions. Fixing it meant adding a missing quantifier to the generated verification conditions, which increased the verification times of the following properties:

#7, #10, #17, #18, #23, #24, #25, #32, #33, #34, #65, #69, #70

Having fixed the bug, we found that our proofs of properties #8 and #54 were invalid. We revisited those problems to fix their proofs, and in doing so found that we cound significantly reduce their necessary lemmas and verification times.

We found that we had mistakenly prevented calls from unrolling in the verification conditions for properties #6, #21, #22, and #31: when that was fixed, we found that they no-longer required any lemmas. For properties #22 and #31, this change increased the verification time, but we consider that to be well worth the reduced manual effort.

We filled in the results for properties #9 and #79, which were unfinished at the time of our submission, and we added a brief description of the effort that went into the case study to the text (extended to cover the time spent on these updates).

Finally, since none of the updated proofs for properties #6, #8, #22, or #23 need to define ≤ as a helper relation any more, we removed the "Extra ops." column from the table.

We have added a description of how EPRCheck generates the multi-function inductive verification conditions to the appendix.

## Raft Case Study

We have added a summary of the details on our Raft case study which we had provided in the revision plan. For completeness, we added the provided details in full to the appendix. We performed a small-scale performance evaluation of our implementation, as compared with the well-known ETCD Raft implementation. This will be replicable in our artifact.

# Related Work

We moved the figure that compares IVy and EPRCheck code examples to the appendix.

# Proofs

We have included proofs of the paper's theorems in the appendix.

# Conclusion

We have added a conclusion section.
