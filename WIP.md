WIP
===

_By: Matthew Gross_

Here, we will explain the objectives and topics of our research, as well as present initial findings and define a general scope as to what is being analyzed.

Topic (C++)
===========

Static_if: a generalized compile-time conditional facility. The symantics of static_if will be similar in nature to the pre-existing, conventional if in that a predicate condition will be stated and two outcomes will be listed. The first, to be executed upon confirmation of a predicate match, is required. The second outcome may be listed using the else keyword. If no listing is made for the else condition, then an empty condition will be implied. The evaluation of the predicate and selection of the ensuing code sequence to execute, however, is proposed to be perfromed during compilation time, rather than run time, when using static_if. To guarantee evaluation, the predicate type passed to static_if must be analogous (and convertible) to a bool type. In order to ensure the ability of the predicate's evaluation at compile time, it must be a constant expression. Just as with normal if statements, nesting of loops, as well as else if statements would be permitted.

The result of this implementation would mean that, at compile time, only one section of code (rather than both) would be required to be tokenized, compiled, and run. The section to which the predicate evaluation does not apply would simply be ignored by the compiler.

In the ensuing analysis, we will discuss both the benefits and repurcussions of such an addition to the C++11 language. This will provide additional weigh-in to a proposal currently up for adoption by the C++ Standards Committee.


Team Members
============
* Matt Bubernak
* Pending

Expected Sources
================
* http://open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3322.pdf (Proposal)
* http://open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3613.pdf (Bjarne Stroustrup's Response)
* http://faculty.cs.niu.edu/~mcmahon/CS241/Notes/compile.html (C++ Compilation Process)
* http://en.wikipedia.org/wiki/C_preprocessor (Preprocessor Schematics)

Code Examples
=============

The simple, standard syntax and implementation, as seen in the initial proposal:

~~~~~~~~~~~~~~~~
1| static_if( predicate ) {
2| body 1
3| }
4| else {
5| body 2
6| }
~~~~~~~~~~~~~~~~

static_if implementation within classes and templates:

~~~~~~~~~~~~~~~~
1| template< class T >
2| class C
3| {
4|  void common( ) { ... }

6|  static_if( has_property1<T>() ) {
7|    void f1( ) { ... }
8|  }

10| static_if( has_property2<T>() ) {
11|   void f2( ) { ... }
12| }
13| else {
14|   void f2( ) = delete;
15| }
16|};
~~~~~~~~~~~~~~~~

static_if to compute factorials more efficiently:

~~~~~~~~~~~~~~~~
1| template <unsigned long n>
2| struct factorial {
3|  static if (n <= 1) {
4|    enum : unsigned long { value = 1 };
5|  } else {
6|    enum : unsigned long { value = factorial<n - 1>::value * n };
7|  }
8| };
~~~~~~~~~~~~~~~~


Work Status
===========

Currently, progress over the writing is minimal. Effort has been garnished toward gathering all required resources, assembling a team, and solidifying a coherent and relative topic.
