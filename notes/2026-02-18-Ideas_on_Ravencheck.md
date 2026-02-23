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
It's not a research problem, but it is worth to fix this.

3. IMO, the way of Ravencheck performs inductive verification is not intuitive. More specifically, it is quite different from the high-level pencil-and-paper proof. For example, let's try to prove `sub(sub(i, j), k) == sub(i, add(j, k))`, TIP #9. A definitions of add and sub are below

```
#[define]
#[recursive]
fn add(a: Nat, b: Nat) -> Nat {
    match a {
        Nat::Z => b,
        Nat::S(a_minus) => Nat::S(Box::new(add(*a_minus,b))),
    }
}
#[define]
#[recursive]
fn sub(x: Nat, y: Nat) -> Nat {
    match x {
        Z => Z,
        S(x_min) => match y {
            Z => x,
            S(y_min) => sub(x_min, y_min),
        }
    }
}
```

What Ravencheck does for inductive verification is to unroll them by the definitions, and compare them. For TIP #9 that we want to prove here, Ravencheck does like below.
```
∀(i,j,k).

let i_j = match i {
    Z => Z,
    S(i_min) => match j {
        Z => i,
        S(j_min) => sub(i_min, j_min),
    }
};

let left = match i_j {
    Z => Z,
    S(i_j_min) => match k {
        Z => i_j,
        S(k_min) => sub(i_j_min, k_min),
    }
};

let j_k = match j {
    Z => k,
    S(j_min) => S(add(j_min, k)),
};

let right = match i {
    Z => Z,
    S(i_min) => match j_k {
        Z => i,
        S(j_k_min) => sub(i_min, j_k_min),
    }
};

left == right
```

What I'm expecting is like below
```
sub(sub(i, j), k) == sub(i, add(j, k))

For case S(i_m) = i, S(j_m) = j, S(k_m) = k, the property would be

sub(sub(S(i_m), S(j_m)), S(k_m)) == sub(S(i_m), add(S(j_m), S(k_m)))
```
To find missing instantiations and follow the proof steps of Ravencheck, I need to unroll all the terms, and it's not that intuitive. I think starting from `sub(sub(S(i_m), S(j_m)), S(k_m)) == sub(S(i_m), add(S(j_m), S(k_m)))` matches better with some kind of high-level view.


## Future directions

1. Currently, Ravencheck requires users to find missing instantiations (e.g., adding let _ = ...) due to its underlying theory (EEPR, relational abstraction and partial function semantics). I believe that finding missing lemmas is a meaningful part of the proof process, but finding missing instantiations is just annoying and tedious. In the future, I want to completely eliminate this burden. I'm somewhat afraid that we might need to come up with different theories, but I'm open to exploring different theoretical foundations—even if it means sacrificing some automation or strict decidability—so that users can focus solely on the logic of the proof.

2. Current verification process of Ravencheck is quite different from handwritten proofs. To verify complex properties, users often have to mentally follow the tool's mechanical steps, such as expanding every match case and unrolling recursion. This is difficult and not intuitive. I want to bridge this gap so that if a property makes sense in a high-level, handwritten proof, it should also be easily verifiable in Ravencheck without the user needing to trace low-level execution details.

3. I know it's a big open problem, but I want to lower the manual efforts(e.g. finding inductive invariants, lemmas)during verification process. Ideally, the only thing we require to the user is propery specifying the property that you want to verify; user don't need to find inductive lemmas or invariants for verification.

4. Finding missing instantiations is a big problem in current version of Ravencheck. When we try to prove complex inductive properties, it is often the case. However, what if we do not delve into inductive problems, and do some regular verification with #[declare] and #[assume], such as case study 6.2 and 6.3 of our OOPSLA paper, maybe automatically inferring axioms for uninterpreted functions would be more useful in this case..?

4.1 A sub-question of #4: What benchamrks should we take for quantifier instantiation for non-inductive verification problems? Maybe I can get some ideas from Counterexample Guided Instantiation paper at OOPSLA23.

4.2 If we want to infer axioms for uninterpreted functions, what additional inputs to an axiom synthesizer should be? Just feeding uninterpreted functions to the synthesizer is too difficult problem to the synthesizer, we need additional hints to guide the synthesizer's search. What kind of hints should we provide? Natrual language descriptions? ....