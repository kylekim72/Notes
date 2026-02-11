## Meeting minutes(2026-02-11)

Will sync again at 2026-02-13 Friday.

## Current status

Ravencheck can map axiom to corresponding function in Rust code. For example, if we write `add_z_right` in our code like below
```
#[annotate]
#[inductive(a: Nat)]
fn add_z_right() -> bool {
    add(a, Nat::Z) == a
}
```
After verifying this lemma is valid, it will show like below
```
SMT Axiom [add_z_right]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ x_a F_Nat__Z____ xn_7)))))
```
In addition, for each valid subcase, it shows the lemmas that is used in that subcase.

## To-do lists

1. Currently, Ravencheck SMT axioms in SMT encoding, which is difficult to get what it means. Translate this into logical expression. For example, the axiom `add_z_right` at the bove block could be written as $$\forall x_a, \forall xn_7 \in \mathbb{N}, \quad \text{add}(x_a, 0, xn_7) \implies xn_7 = x_a$$

2. Output raw model, and replace constants as much as I can(also sorts too)
For example, if we see something linke `(= x UI_Nat!val!3), (= y UI_Nat!val!0), (= z UI_Nat!val!7)` in raw model, replace `UI_Nat!val!3` with `x`, `UI_Nat!val!0` with `y`, `UI_Nat!val!7` with `z`, respectively.
In addition, we don't know what `UI_Nat` is. Replace the types used in raw model with the types in Rust code. In this case, we need to replace `UI_Nat` with `Nat`.