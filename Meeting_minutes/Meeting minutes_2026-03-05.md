## Meeting minutes: 2026-03-05

1. Currently, Ravencheck does not allows user to write a proof. For example, when I'm proving TIP benchmarks in F*, I can write a manual proof like `match a with ...`. Ravencheck just unrolls the recursive calls in the property and check LHS == RHS, which makes proving a property more difficult.

The above is a flaw of Ravencheck and an advantage of F*. In Ravencheck, we are not allowed to provide a proof and must simply see what Ravencheck does, which makes discharging verification conditions (VCs) more difficult. In contrast, in F* we can provide a manual proof, allowing us to guide the proof more easily. However, because F* is a fully dependent typed language, it cannot guarantee decidability, whereas Ravencheck can. Due to this undecidability, F* cannot provide counterexamples. This is an advantage of Ravencheck and a limitation of F*.

How can we combine only the good parts of both? This would be a core question of my current research. In this case, we need to provide a way to user can guide a inductive proof in Ravencheck.

2. To enhance the understanding of how dependent type system works, define a toy coreML syntax and implement a type checking rule based on inference rules in Liquid Types paper at PLDI08. OCaml is recommended language to implement this type checking algorithm.