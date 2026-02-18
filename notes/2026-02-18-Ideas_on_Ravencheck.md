## Some thoughts on Ravencheck

What I felt during working with Ravencheck(proving TIP benchmarks, implementing counterexample generation), it is cool but it has some distinct drawbacks. Although my experience with Ravencheck leans toward to inductive proof, I believe some of its limitations can be applied to general situtations as well(e.g. verifcation with #[verify], #[declare] and #[assume]).

1. IMO, the biggest obstacle to use Ravencheck is finding missing instantiations. Since Ravencheck leverages relational abstraction and partial function semantics to ensure decidability, it allows some terms to be undefined, leading to verification failure. To find missing instantiations, in inductive proof, an user need to unroll all the recursive definitions and do deep dive into match statements of recursive functions. I'm guessing it would be more difficult in non-inductive case, since it is unclear to what should I follow to find missing instantiations.

2. Connected to #1, the way to add missing instantiations is not intuitive. For example, to prove commutativity of add, I need to instantiate `add(S(b_m), a_m)` (assume b = S(b_m), a = S(a_m)) with some lemmas. The way of adding instatiation of `add(S(b_m), a_m)` looks like below

```
#[annotate]
#[inductive(a: Nat, b: Nat)]
fn add_commutative() -> bool {
    let _ = match b {
        Nat::Z => Nat::Z,
        Nat::S(b_m) => match a {
            Nat::Z => Nat::Z,
            Nat::S(a_m) => add(Nat::S(b_m), a_m),
        }
    };

    add(a,b) == add(b,a)
}
```

Although I'm currently not familiar with other verification tools(e.g. F*), I'm sure that this is not iutuitive.

3.(Not completed yet... need more complex example) The way of Ravencheck performs inductive verification is not intuitive. More specifically, it is quite different from the high-level pencil-and-paper proof. For example, let's try to prove commutativity of add. A definitions of add is below

```
#[define]
#[recursive]
fn add(a: Nat, b: Nat) -> Nat {
    match a {
        Nat::Z => b,
        Nat::S(a_minus) => Nat::S(Box::new(add(*a_minus,b))),
    }
}
```

For the last subcase(a = S(a_m) and b = S(b_m)), Ravencheck behaves like below to prove add_commutative
```
add(a,b) = match a {
    Z => b,
    S(a_m) => S(add(a_m, b)) == add(b,a) = match b{
        Z => a,
        S(b_m) => S(add(b_m, a))
    }
}
```
It dives into match statements and compare the final ouput. In this casewe want to prove `S(add(a_m, b)) == S(add(b_m, a))` after unrolling.


## Future directions

1. Currently, Ravencheck requires users to find missing instantiations (e.g., adding let _ = ...) due to its underlying theory (EEPR, relational abstraction and partial function semantics). I believe that finding missing lemmas is a meaningful part of the proof process, but finding missing instantiations is just annoying and tedious. In the future, I want to completely eliminate this burden. I'm somewhat afraid that we might need to come up with different theories, but I'm open to exploring different theoretical foundations—even if it means sacrificing some automation or strict decidability—so that users can focus solely on the logic of the proof.

2. Current verification process of Ravencheck is quite different from handwritten proofs. To verify complex properties, users often have to mentally follow the tool's mechanical steps, such as expanding every match case and unrolling recursion. This is difficult and not intuitive. I want to bridge this gap so that if a property makes sense in a high-level, handwritten proof, it should also be easily verifiable in Ravencheck without the user needing to trace low-level execution details.