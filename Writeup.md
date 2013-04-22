C++ Static-if Implementation - A Critical Analysis
==================================================

_Authors: Matthew Gross & Matthew Bubernak_


**Introduction** 

In late 2011, a new generalized compile-time conditional facility was proposed by Walter E. Brown: static_if. The proposed symantics of static_if would be similar in nature to the pre-existing, conventional "if" in that a predicate condition would be stated and two outcomes would be accordingly listed. The motivation behind this change is centered around the thought process of creating a more efficient and lightweight runtime space. By performing, essentially, pre-compile if statements, potentially unreachable code can be discarded and only the appropriate predicate result will be included in the final process compile code.

The simple, standard syntax and implementation, as seen in the initial proposal:

~~~~~~~~~~~~~~~~
1| static_if( predicate ) {
2| body 1
3| }
4| else {
5| body 2
6| }
~~~~~~~~~~~~~~~~

**Symantics & Attributes**

Two possible outcomes may be provided in regards to the evaluation of the static_if predicate. The first, to be executed upon confirmation of a predicate match, is required. The second outcome may be listed using the else keyword. If no listing is made for the else condition, then an empty condition will be implied. The evaluation of the predicate and selection of the ensuing code sequence to execute, however, is proposed to be perfromed during compilation time, rather than run time, when using static_if. To guarantee evaluation, the predicate type passed to static_if must be analogous (and convertible) to a bool type. In order to ensure the ability of the predicate's evaluation at compile time, it must be a constant expression. Just as with normal if statements, nesting of loops, as well as else if statements would be permitted.

Upon the evaluation of the predicate, the correct resulting path is selected and then put in queue for compilation. The other path is discarded. In this case, we see an example of non-strict evaluation in that the unused path is discarded regardless of correctness or value. Only the correct path is then evaluated during compilation. In addition, this signifies that, at compile time, only one section of code (rather than both) would be required to be tokenized, compiled, and run. The section to which the predicate evaluation does not apply would simply be ignored by the compiler (again, showing a non-strict evaluational nature).

In terms of scoping, this implementation would adhear to the scopes of namespace, class, and block. There is even the possibility this could extend to wherever c++ permits braces. It would be expected that static_if would adhere to similar scoping as static_assert. This scoping is significant in broadening the number of data types that the static_if can evaluate, as well as the allowing static_if to span a broader range of application. An example of this broader scope can be seen within implementation using templates:

In a hypothetical situation, one may have a series of constexpr function templates, each resembling the below code: 

~~~~~~~~~~~~~~~~
1| template< class T >
2| constexpr bool
3| has_property_n( ) { return ...; }
~~~~~~~~~~~~~~~~

In this situation, there is a class template c that has a single type parameter, and the implementation of all the member functions of C depend on the truth values of the property inquiry functions. Currently in c++, n amount of property inquiries would require up to n non-type bool template parameters which furthermore provide up to 2^n specializations. Much of this code is redundant. 

Static_if statements are the solutions to desired functionality. Instead of tag dispatch, code duplication, and extra template parameters, one can solve the problem with an if statement structure as presented below.  

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

Furthermore, in regards to scoping, static_if's functionality is comparable to the c++ preprocessor's use of #if and #endif. This mechanism for conditional compilation follows similar semantics to the proposed static_if construct. The issue is that preprocessing operates in a scope that does not allow for the evaluation of constant expressions. Static_if would theoretically change this preprocessing system by including constant expressions in the scope of the if statement. 

An added advantage to the use of static_if is the increased efficiency of functional overrides. Below, we see an example of three functional overrrides:

~~~~~~~~~~~~~~~~
1|  template <class BidirectionalIterator>
2|  inline void
3|  evolve(BidirectionalIterator first, BidirectionalIterator last) {
4|  evolve(first, last,
5|  typename iterator_traits<BidirectionalIterator>::iterator_category());
6|  }
8|  template <class BidirectionalIterator>
9|  void evolve(BidirectionalIterator first, BidirectionalIterator last,
10| bidirectional_iterator_tag) {
11| // more generic, but less efficient algorithm
12| }
14| template <class RandomAccessIterator>
15| void evolve(RandomAccessIterator first, RandomAccessIterator last,
16| random_access_iterator_tag) {
17| // more efficient, but less generic algorithm
18| }
~~~~~~~~~~~~~~~~

This can be accordingly translated to the following static_if structure:

~~~~~~~~~~~~~~~~
1|  template <class Iterator>
2|  inline void
3|  evolve(Iterator first, Iterator last)
4|  static_if( is_bidirectional<Iterator>() ) {
5|  // more generic, but less efficient algorithm
6|  }
7|  elseif( is_random_access<Iterator>() ) {
8|  // more efficient, but less generic algorithm
9|  }
10| else {
11| issue_diagnostic(...);
12| }
~~~~~~~~~~~~~~~~

**Community Response**



**Conclusion & Closing Gestures**

