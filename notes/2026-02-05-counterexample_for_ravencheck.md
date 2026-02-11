## Intro

Currently, Ravencheck outputs only binary yes/no answers for verification conditions. We need a counterexample that gives useful information to user to succeed verification. In this note, we will focus on inductive problems.

For complex inductive verification problems, just specifying verification conditions and recursive definitions is insufficient. It might require some intermediate lemmas to make a successful proof. In addition to this, Ravencheck has another problem: although all required lemmas are ready-to-go, it might not be used during verification due to instantiation problem. To ensure decidability and predictability, Ravencheck leverages relational abstraction and partial function semantics. Although these techniques allows us to stay in decidable & predictable world, it allows some terms to be undefined, which hinders the use of relevant lemmas. Therefore, we have 2 main problems: find missing lemmas & instantiations.

In this note, I will talk about what Ravencheck outputs for counterexample currently(especially, for inductive verification such as TIP benchmarks), and how can we improve this into a more readable format. I will start with simple property that only requires 1 lemma to be proved.

## Current approach

Ideal soultion for these problems would be suggesting missing lemmas & instantiations(respectively), but it might be difficult to reach out there. Therefore, we suggest an alternative solutions for ideal solution. First, to find missing lemmas, I want to show the final LHS and RHS when it fails for certain case. For example, let's say we want to prove `add(S(a), b) == add(a, S(b))` and it fails at a = Z and b = S(b_m) case. In this case, we want to show "In a = Z and b = S(b_m), verification fails and LHS is ... and RHS is ...". It will help to figure out what lemmas are missing. In addition to this, we also want to show what lemmas are used during verification process. If certain lemma is declared in code but not used in verification, we can infer that there is an instantiation problem.


## Current status

Ravencheck outputs function's name. Below is a part of Ravencheck's output when it fails..
```
Declared add as absrel F_add____
SMT Axiom [Rel-Fun for add]: (forall ((xn_0 UI_Nat) (xn_1 UI_Nat)) (forall ((xn_3 UI_Nat)) (forall ((xn_5 UI_Nat)) (or (= xn_3 xn_5) (or (not (F_add____ xn_0 xn_1 xn_5)) (not (F_add____ xn_0 xn_1 xn_3)))))))
Declared substruct::<Nat> as relation F_substruct__UI_Nat__ with 2 inputs
SMT Axiom [add_z_left]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ F_Nat__Z____ x_a xn_7)))))

SMT Axiom [add_z_right]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ x_a F_Nat__Z____ xn_7)))))

SMT Axiom [add_s_right]: (forall ((x_a UI_Nat) (x_b UI_Nat)) (forall ((xn_14 UI_Nat)) (forall ((xn_15 UI_Nat)) (forall ((xn_16 UI_Nat)) (forall ((xn_17 UI_Nat)) (or (or (or (or (= xn_15 xn_17) (not (F_Nat__S____ x_b xn_14))) (not (F_add____ x_a xn_14 xn_15))) (not (F_add____ x_a x_b xn_16))) (not (F_Nat__S____ xn_16 xn_17))))))))

SMT Axiom [add_move_s]: (forall ((x_a UI_Nat) (x_b UI_Nat)) (forall ((xn_14 UI_Nat)) (forall ((xn_15 UI_Nat)) (forall ((xn_16 UI_Nat)) (forall ((xn_17 UI_Nat)) (or (or (or (or (= xn_15 xn_17) (not (F_Nat__S____ x_a xn_14))) (not (F_add____ xn_14 x_b xn_15))) (not (F_Nat__S____ x_b xn_16))) (not (F_add____ x_a xn_16 xn_17))))))))

SMT Axiom [Substruct_Nat_0]: (forall ((xn_0 UI_Nat)) (F_substruct__UI_Nat__ xn_0 xn_0))

SMT Axiom [Substruct_Nat_1]: (forall ((xn_0 UI_Nat) (xn_1 UI_Nat) (xn_2 UI_Nat)) (or (F_substruct__UI_Nat__ xn_0 xn_2) (or (not (F_substruct__UI_Nat__ xn_1 xn_2)) (not (F_substruct__UI_Nat__ xn_0 xn_1)))))

SMT Axiom [Substruct_Nat_2]: (forall ((xn_0 UI_Nat) (xn_1 UI_Nat)) (or (= xn_0 xn_1) (or (not (F_substruct__UI_Nat__ xn_1 xn_0)) (not (F_substruct__UI_Nat__ xn_0 xn_1)))))

SMT Axiom [Substruct_Nat_3]: (forall ((xn_1 UI_Nat)) (or (forall ((xn_3 UI_Nat)) (= (F_substruct__UI_Nat__ xn_3 xn_1) (= xn_3 xn_1))) (distinct F_Nat__Z____ xn_1)))

SMT Axiom [Substruct_Nat_4]: (forall ((xn_0 UI_Nat)) (forall ((xn_2 UI_Nat)) (or (and (distinct xn_0 xn_2) (F_substruct__UI_Nat__ xn_0 xn_2)) (not (F_Nat__S____ xn_0 xn_2)))))

SMT VC: (exists ((x_a UI_Nat) (x_b UI_Nat)) (exists ((xn_11 UI_Nat)) (exists ((xn_94 UI_Nat)) (exists ((xn_95 UI_Nat)) (and (and (and (exists ((xn_20 UI_Nat)) (exists ((xn_105 UI_Nat)) (exists ((xn_106 UI_Nat)) (and (and (and (and (distinct xn_95 xn_106) (forall ((xn_29 UI_Nat) (xn_30 UI_Nat)) (or (forall ((xn_56 UI_Nat)) (forall ((xn_57 UI_Nat)) (or (or (= xn_56 xn_57) (not (F_add____ xn_29 xn_30 xn_56))) (not (F_add____ xn_30 xn_29 xn_57))))) (or (and (= xn_30 x_b) (= xn_29 x_a)) (or (not (F_substruct__UI_Nat__ xn_30 x_b)) (not (F_substruct__UI_Nat__ xn_29 x_a))))))) (F_Nat__S____ xn_20 x_b)) (F_add____ xn_20 x_a xn_105)) (F_Nat__S____ xn_105 xn_106))))) (F_Nat__S____ xn_11 x_a)) (F_add____ xn_11 x_b xn_94)) (F_Nat__S____ xn_94 xn_95))))))
```

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

