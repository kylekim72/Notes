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
After verifying this lemma is valid, Ravencheck will ouput like below
```
SMT Axiom [add_z_right]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ x_a F_Nat__Z____ xn_7)))))
```
In addition, for each valid subcase, it shows the lemmas that is used in each subcase.

## To-do lists

1. Currently, Ravencheck SMT axioms in SMT encoding, which is difficult to get what it means. Translate this into logical expression. For example, the axiom `add_z_right` at the bove block could be written as $$\forall x_a, \forall xn_7 \in \mathbb{Nat}, \quad \text{add}(x_a, 0, xn_7) \implies xn_7 = x_a$$

2. Output raw model, and replace constants as much as I can(also sorts too)
For example, if we see something linke `(= x UI_Nat!val!3), (= y UI_Nat!val!0), (= z UI_Nat!val!7)` in raw model, replace `UI_Nat!val!3` with `x`, `UI_Nat!val!0` with `y`, `UI_Nat!val!7` with `z`, respectively.
In addition, we don't know what `UI_Nat` is. Replace the types used in raw model with the types in Rust code. In this case, we need to replace `UI_Nat` with `Nat`.

The final goal of this task is to map this low-level variables(in SMT expressions) to high-level variables(in Rust code that user have written), but let's start with simple one.

## 2026-02-13 Friday status

1. Completed implementation of the code that print axioms in more readable way. The code block below shows the original outputs of printing axioms(SMT Axiom []: ...) and current outputs(Infix notation: ...).
```
SMT Axiom [add_z_left]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ F_Nat__Z____ x_a xn_7)))))

Infix notation: (∀ x_a: Nat. (∀ xn_7: Nat. (xn_7 == x_a || !add(Z, x_a) == xn_7)))

SMT Axiom [add_z_right]: (forall ((x_a UI_Nat)) (forall ((xn_7 UI_Nat)) (or (= xn_7 x_a) (not (F_add____ x_a F_Nat__Z____ xn_7)))))

Infix notation: (∀ x_a: Nat. (∀ xn_7: Nat. (xn_7 == x_a || !add(x_a, Z) == xn_7)))

SMT Axiom [add_s_right]: (forall ((x_a UI_Nat) (x_b UI_Nat)) (forall ((xn_14 UI_Nat)) (forall ((xn_15 UI_Nat)) (forall ((xn_16 UI_Nat)) (forall ((xn_17 UI_Nat)) (or (or (or (or (= xn_15 xn_17) (not (F_Nat__S____ x_b xn_14))) (not (F_add____ x_a xn_14 xn_15))) (not (F_add____ x_a x_b xn_16))) (not (F_Nat__S____ xn_16 xn_17))))))))

Infix notation: (∀ x_a: Nat, x_b: Nat. (∀ xn_14: Nat. (∀ xn_15: Nat. (∀ xn_16: Nat. (∀ xn_17: Nat. ((((xn_15 == xn_17 || !S(x_b) == xn_14) || !add(x_a, xn_14) == xn_15) || !add(x_a, x_b) == xn_16) || !S(xn_16) == xn_17))))))

SMT Axiom [add_move_s]: (forall ((x_a UI_Nat) (x_b UI_Nat)) (forall ((xn_14 UI_Nat)) (forall ((xn_15 UI_Nat)) (forall ((xn_16 UI_Nat)) (forall ((xn_17 UI_Nat)) (or (or (or (or (= xn_15 xn_17) (not (F_Nat__S____ x_a xn_14))) (not (F_add____ xn_14 x_b xn_15))) (not (F_Nat__S____ x_b xn_16))) (not (F_add____ x_a xn_16 xn_17))))))))

Infix notation: (∀ x_a: Nat, x_b: Nat. (∀ xn_14: Nat. (∀ xn_15: Nat. (∀ xn_16: Nat. (∀ xn_17: Nat. ((((xn_15 == xn_17 || !S(x_a) == xn_14) || !add(xn_14, x_b) == xn_15) || !S(x_b) == xn_16) || !add(x_a, xn_16) == xn_17))))))
```

2. Now Ravencheck outputs Raw model and it replaces uninterpreted types with the types in Rust code. For example, if the raw model output like below,
```
Raw model
((define-fun F_special_recursive____ () Bool false) (define-fun F_Nat__Z____ () UI_Nat (as @UI_Nat_0 UI_Nat)) (define-fun F_Nat__S____ (($x1 UI_Nat) ($x2 UI_Nat)) Bool (or (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_2 UI_Nat) $x2)) (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_1 UI_Nat) $x2)) (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2)))) (define-fun F_substruct__UI_Nat__ (($x1 UI_Nat) ($x2 UI_Nat)) Bool (and (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_2 UI_Nat) $x2))) (not (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2))) (not (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2))))) (define-fun F_add____ (($x1 UI_Nat) ($x2 UI_Nat) ($x3 UI_Nat)) Bool (or (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2) (= (as @UI_Nat_4 UI_Nat) $x3)) (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_1 UI_Nat) $x2) (= (as @UI_Nat_3 UI_Nat) $x3)))))
```
it replaces SMT solver's types with the types in Rust code(in this case, UI_Nat is corresponding to Nat in Rust code). 
```
Raw model after type replace
((define-fun F_special_recursive____ () Bool false) (define-fun F_Nat__Z____ () Nat (as @Nat_0 Nat)) (define-fun F_Nat__S____ (($x1 Nat) ($x2 Nat)) Bool (or (and (= (as @Nat_4 Nat) $x1) (= (as @Nat_2 Nat) $x2)) (and (= (as @Nat_2 Nat) $x1) (= (as @Nat_1 Nat) $x2)) (and (= (as @Nat_3 Nat) $x1) (= (as @Nat_4 Nat) $x2)))) (define-fun F_substruct__Nat__ (($x1 Nat) ($x2 Nat)) Bool (and (not (and (= (as @Nat_1 Nat) $x1) (= (as @Nat_0 Nat) $x2))) (not (and (= (as @Nat_1 Nat) $x1) (= (as @Nat_2 Nat) $x2))) (not (and (= (as @Nat_4 Nat) $x1) (= (as @Nat_0 Nat) $x2))) (not (and (= (as @Nat_4 Nat) $x1) (= (as @Nat_3 Nat) $x2))) (not (and (= (as @Nat_3 Nat) $x1) (= (as @Nat_0 Nat) $x2))) (not (and (= (as @Nat_2 Nat) $x1) (= (as @Nat_0 Nat) $x2))) (not (and (= (as @Nat_2 Nat) $x1) (= (as @Nat_4 Nat) $x2))) (not (and (= (as @Nat_1 Nat) $x1) (= (as @Nat_4 Nat) $x2))) (not (and (= (as @Nat_2 Nat) $x1) (= (as @Nat_3 Nat) $x2))) (not (and (= (as @Nat_1 Nat) $x1) (= (as @Nat_3 Nat) $x2))))) (define-fun F_add____ (($x1 Nat) ($x2 Nat) ($x3 Nat)) Bool (or (and (= (as @Nat_2 Nat) $x1) (= (as @Nat_4 Nat) $x2) (= (as @Nat_4 Nat) $x3)) (and (= (as @Nat_3 Nat) $x1) (= (as @Nat_1 Nat) $x2) (= (as @Nat_3 Nat) $x3)))))
```
In addition to this, it also outputs cleaned-up result to show the relations and interactions between those constants and relations in the model.

```
const F_special_recursive____ = false;
const F_Nat__Z____ = @Nat_0;

fn F_Nat__S____($x1, $x2) {
  Case 1:  @Nat_4 == $x1 && @Nat_2 == $x2
  Case 2:  @Nat_2 == $x1 && @Nat_1 == $x2
  Case 3:  @Nat_3 == $x1 && @Nat_4 == $x2
}

fn F_substruct__Nat__($x1, $x2) {
  Constraint: @Nat_1 == $x1 && @Nat_0 == $x2 && @Nat_1 == $x1 && @Nat_2 == $x2 && @Nat_4 == $x1 && @Nat_0 == $x2 && @Nat_4 == $x1 && @Nat_3 == $x2 && @Nat_3 == $x1 && @Nat_0 == $x2 && @Nat_2 == $x1 && @Nat_0 == $x2 && @Nat_2 == $x1 && @Nat_4 == $x2 && @Nat_1 == $x1 && @Nat_4 == $x2 && @Nat_2 == $x1 && @Nat_3 == $x2 && @Nat_1 == $x1 && @Nat_3 == $x2
}

fn F_add____($x1, $x2, $x3) {
  Case 1:  @Nat_2 == $x1 && @Nat_4 == $x2 && @Nat_4 == $x3
  Case 2:  @Nat_3 == $x1 && @Nat_1 == $x2 && @Nat_3 == $x3
}
```

3. I've implemented to dump the SMT query that Ravencheck throws to the SMT solver as `reproduce.smt2`. After Ravencheck's verification, user can directly interact with the solver such as `cvc5 reproduce.smt2` like below.

```
➜  examples git:(main) ✗ cvc5 reproduce.smt2
sat
(
; cardinality of UI_Nat is 5
; rep: (as @UI_Nat_0 UI_Nat)
; rep: (as @UI_Nat_1 UI_Nat)
; rep: (as @UI_Nat_2 UI_Nat)
; rep: (as @UI_Nat_3 UI_Nat)
; rep: (as @UI_Nat_4 UI_Nat)
(define-fun F_special_recursive____ () Bool false)
(define-fun F_Nat__Z____ () UI_Nat (as @UI_Nat_0 UI_Nat))
(define-fun F_Nat__S____ (($x1 UI_Nat) ($x2 UI_Nat)) Bool (or (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_2 UI_Nat) $x2)) (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_1 UI_Nat) $x2)) (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2))))
(define-fun F_substruct__UI_Nat__ (($x1 UI_Nat) ($x2 UI_Nat)) Bool (and (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_2 UI_Nat) $x2))) (not (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_4 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2))) (not (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_0 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2))) (not (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2))) (not (and (= (as @UI_Nat_1 UI_Nat) $x1) (= (as @UI_Nat_3 UI_Nat) $x2)))))
(define-fun F_add____ (($x1 UI_Nat) ($x2 UI_Nat) ($x3 UI_Nat)) Bool (or (and (= (as @UI_Nat_2 UI_Nat) $x1) (= (as @UI_Nat_4 UI_Nat) $x2) (= (as @UI_Nat_4 UI_Nat) $x3)) (and (= (as @UI_Nat_3 UI_Nat) $x1) (= (as @UI_Nat_1 UI_Nat) $x2) (= (as @UI_Nat_3 UI_Nat) $x3))))
)
```
