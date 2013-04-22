C++ Static-if Implementation - A Critical Analysis
==================================================

_Authors: Matthew Gross & Matthew Bubernak_


**Introduction** 

In late 2011, a new generalized compile-time conditional facility was proposed by Walter E. Brown: static_if. The proposed symantics of static_if would be similar in nature to the pre-existing, conventional "if" in that a predicate condition would be stated and two outcomes would be accordingly listed. The motivation behind this change is centered around the thought process of creating a more efficient and lightweight runtime space. By performing, essentially, pre-compile if statements, potentially unreachable code can be discarded and only the appropriate predicate result will be included in the final process compile code.

**Symantics & Attributes**

Two possible outcomes may be provided in regards to the evaluation of the static_if predicate. The first, to be executed upon confirmation of a predicate match, is required. The second outcome may be listed using the else keyword. If no listing is made for the else condition, then an empty condition will be implied. The evaluation of the predicate and selection of the ensuing code sequence to execute, however, is proposed to be perfromed during compilation time, rather than run time, when using static_if. To guarantee evaluation, the predicate type passed to static_if must be analogous (and convertible) to a bool type. In order to ensure the ability of the predicate's evaluation at compile time, it must be a constant expression. Just as with normal if statements, nesting of loops, as well as else if statements would be permitted.

Upon the 

**Community Response**

The result of this implementation would mean that, at compile time, only one section of code (rather than both) would be required to be tokenized, compiled, and run. The section to which the predicate evaluation does not apply would simply be ignored by the compiler.

**Conclusion & Closing Gestures**

In the ensuing analysis, we will discuss both the benefits and repurcussions of such an addition to the C++11 language. This will provide additional weigh-in to a proposal currently up for adoption by the C++ Standards Committee.
