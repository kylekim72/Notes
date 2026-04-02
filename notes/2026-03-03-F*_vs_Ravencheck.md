## Proving TIP benchmarks: Ravencheck VS F*

Recently, I'm learning F* and it's really fun! While reading F* book to enhance my understanding, I've picked few TIP benchmarks from our Ravencheck paper and proved them using F*.

Here is the link for TIP #8 and #9 written in Ravencheck and F*.

TIP #8 with Ravencheck: https://github.com/cuplv/ravencheck-tip-benchmarks/blob/main/src/isaplanner/prop_08.rs

TIP #9 with Ravencheck: https://github.com/cuplv/ravencheck-tip-benchmarks/blob/main/src/isaplanner/prop_09.rs

TIP #79 with Ravencheck: https://github.com/cuplv/ravencheck-tip-benchmarks/blob/main/src/isaplanner/prop_79.rs

TIP #8 with F*: https://github.com/kylekim72/Fstar_playground/blob/main/TIP_benchmarks/TIP_8.fst

TIP #9 with F*: https://github.com/kylekim72/Fstar_playground/blob/main/TIP_benchmarks/TIP_9.fst

TIP #79 with F*: https://github.com/kylekim72/Fstar_playground/blob/main/TIP_benchmarks/TIP_79.fst

To prove TIP #8, #9, #79 in Ravencheck, we need 7 lemmas & 1 instantiations and 4 lemmas & 2 instantiations, 5 lemmas, respectively. However, in F*, there is no need of additional lemmas to prove TIP #8 and #9.

The need of instantiation is due to the relational abstraction(more specifically, lack of totality). This is acceptable, because Ravencheck is based on EEPR + relational abstraction, and F* is based on dependent type theory, but what about lemmas? What's the key difference between Ravencheck and F* on induction?

## Proof of TIP 9

```
type my_nat =
  | Z : my_nat
  | S : my_nat -> my_nat

let rec sub (x y: my_nat) : my_nat =
  match x with
  | Z -> Z
  | S x' ->
    (match y with
      | Z -> x
      | S y' -> sub x' y')

let rec add (x y: my_nat) : my_nat =
  match x with
  | Z -> y
  | S x' -> S (add x' y)

val tip_nine (a b c: my_nat) : Lemma (ensures sub (sub a b) c == sub a (add b c))

let rec tip_nine a b c =
  match a with
  | Z -> ()
  | S a' ->
    (match b with
      | Z -> ()
      | S b' -> tip_nine a' b' c)
```

### a = Z

Prove `sub(sub(Z, b), c) == sub(Z, (add(b, c)))`

RHS becomes Z by applying definition of sub once. LHS becomes Z by applying definition of sub twice(sub(Z,b) == Z and sub(Z,c) == Z).

### a = S(a')

In this case, we need know the shape of b(whether it is Z or successor of b').

If b is Z, prove `sub(sub(S(a'), Z), c) == sub(S(a'), add(Z, c))`.

LHS: unroll sub(S(a'), Z) to S(a'), then LHS becomes sub(S(a'), c).

RHS: unroll add(Z,c) to c, then RHS becomes sub(S(a'), c).

LHS and RHS are completely equal, thus VC discharges.

If b is S(b'), prove `sub(sub(S(a'), S(b')), c) == sub(S(a'), add(S(b'), c))`

LHS: unroll sub(S(a'), S(b')) to sub(a', b'), then LHS becomes sub(sub(a', b'), c).

RHS: unroll add(S(b'), c) to S(add(b', c)) and unroll sub(S(a'), S(add(b', c))) to sub(a', add(b', c)). Then RHS becomes sub(a', add(b', c)).

In this case, the inductive hypothesis that we gave here `S b' -> tip_nine a' b' c)` discharges VC.

## Idea for Ravencheck

Ravencheck handles recursive functions with 1-step unrolling, and it might make a proof harder because when Ravencheck unrools recursive functions, it loses the information of call. Thus, the idea is to add constraints of the function call.

For example, let's say we want to reason about $\forall x,y. P(add(x,y))$(P is a function). If we unroll add(x,y), it looks like below:

```
P(add(x, y)) = match x{
    Z -> P(y),
    S(x') -> P(S(add(x', y)))
}
```
After unrolling, it loses it's shape before unrolling. So the idea is to remember the information before unrolling like this:
$\forall x,y. x == Z => add(x,y) = y /\ x == S(x') => add(x,y) = $



