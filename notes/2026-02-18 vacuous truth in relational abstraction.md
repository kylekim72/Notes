## Current status

I'm working on implementing a feature that checks vacuous truth of relational abstraction. For example, let's say we want to commutative property of addition, add(a,b) == add(b,a). If we write this property with functions, we can write this as $\forall a,b \  add(a,b) == add(b,a)$. With relatioanl abstraction, this property will be described as $\forall a,b,c,d  \ Rel_{add}(a,b,c) \wedge Rel_{add}(b,a,d) \implies c == d$.
