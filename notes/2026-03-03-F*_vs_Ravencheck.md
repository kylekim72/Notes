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


