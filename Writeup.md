C++ Static_if Implementation - A Critical Analysis
==================================================

**_Authors: Matthew Gross & Matthew Bubernak_**

*Word Count (Excluding Code): 1208*

Collaboration Repository: https://github.com/mattkgross/PrincPaper

**Introduction** 

In late 2011, a new generalized compile-time conditional facility was proposed by Walter E. Brown: *static_if*. The proposed semantics of *static_if* would be similar in nature to the pre-existing, conventional "if" in that a predicate condition would be stated and two outcomes would be accordingly listed. The motivation behind this change is centered around the thought process of creating a more efficient and lightweight runtime space. By performing, essentially, pre-compile if statements, potentially unreachable code can be discarded and only the appropriate predicate result will be included in the final process compile code.

The simple, standard syntax and implementation, as seen in the initial proposal:

~~~~~~~~~~~~~~~~
1| static_if( predicate ) {
2| body 1
3| }
4| else {
5| body 2
6| }
~~~~~~~~~~~~~~~~

**Semantics & Attributes**

Two possible outcomes may be provided in regards to the evaluation of the *static_if* predicate. The first, to be executed upon confirmation of a predicate match, is required. The second outcome may be listed using the else keyword. If no listing is made for the else condition, then an empty condition will be implied. The evaluation of the predicate and selection of the ensuing code sequence to execute, however, is proposed to be performed during compilation time, rather than run time, when using *static_if*. To guarantee evaluation, the predicate type passed to *static_if* must be analogous (and convertible) to a boolean type. In order to ensure the ability of the predicate's evaluation at compile time, it must be a constant expression. Just as with normal if statements, nesting of loops, as well as else if statements, would be permitted.

Upon the evaluation of the predicate, the correct resulting path is selected and then put in queue for compilation. The other path is discarded. In this case, we see an example of non-strict evaluation in that the unused path is discarded regardless of correctness or value. Only the correct path is then evaluated during compilation. In addition, this signifies that, at compile time, only one section of code (rather than both) would be required to be tokenized, compiled, and run. The section to which the predicate evaluation does not apply would simply be ignored by the compiler (again, showing a non-strict evaluative nature).

In terms of scoping, this implementation would adhere to the scopes of namespace, class, and block. There is even the possibility this could extend to wherever C++ permits braces. In a general sense it would be expected that *static_if* would adhere to similar scoping as *static_assert*. This scoping is significant in broadening the number of data types that the *static_if* can evaluate, as well as allowing *static_if* to span a broader range of application. An example of this broader scope can be seen within implementation using templates.

In a hypothetical situation, one may have a series of *constexpr* function templates, each resembling the below code: 

~~~~~~~~~~~~~~~~
1| template< class T >
2| constexpr bool
3| has_property_n( ) { return ...; }
~~~~~~~~~~~~~~~~

In this situation, there is a class template, *C*, that has a single type parameter. The implementation of all the member functions of *C* depend on the truth values of the property inquiry functions. Currently, in C++, *n* amount of property inquiries would require up to *n* non-type boolean template parameters which furthermore provide up to *2^n* specializations. Much of this code is redundant. 

*Static_if* statements are the solutions to desired functionality. Instead of tag dispatch, code duplication, and extra template parameters, one can solve the problem with an if statement structure as presented below.  

*Static_if* implementation within classes and templates:

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

Furthermore, in regards to scoping, *static_if*'s functionality is comparable to the C++ preprocessor's use of *#if* and *#endif*. This mechanism for conditional compilation follows similar semantics to the proposed *static_if* construct. The issue is that preprocessing operates in a scope that does not allow for the evaluation of constant expressions. *Static_if* would theoretically change this preprocessing system by including constant expressions in the scope of the if statement. 

An added advantage to the use of *static_if* is the increased efficiency of functional overrides. Below, we see an example of three functional overrrides:

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

This can be accordingly translated to the following *static_if* structure:

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

From a parsing standpoint, the *static_if* implementation proves friendlier on the tokenizer, as only one template must be declared and its scoping extends to all three predicate branches of the *static_if* statement. Additionally, only one function must be type casted and tokenized upon compile time - proving to be even more efficient.

**Community Response**

To this point, the benefits of *static_if* statements have been outlined. However, there are valid claims that have been brought up to challenge the perceived advantages of the *static_if* statement. They center around three recurring design problems: conditionally included statements, conditionally defined interfaces, and constrained templates.

The selective inclusion of statements in a translation unit composes conditional compilation. In contrast to normal if statements, *static_if*'s statement braces do not indicate a new scope. In fact, any declarations within them are simply included in whatever scope contains the static_if block. For this reason, confusion is sure to arise in regards to scoping if both static and non-static if statements are used. 

Another advertised advantage to *static_if* is its use as an alternative to traditional ways of compile-time source manipulation. The issue here is that there is no way to fully remove the use of *#ifdef* and macros. Instead, the result will be a mixture of old and new techniques. In order to avoid the use of *static_if* and preprocessor tricks, one would probably end up having to move on to *static_for*, *static_while*, etc., and the result would be a more low-leveled C++ emergence. 

*Static_if* statements could also present issues with template functions. Using *static_if* inside a template may prevent the compiler from performing even the simplest checks on template definitions. *Static_if* would essentially fail to improve the speed of checking template arguments or improving diagnostics. The compiler is unable to parse the branches of the compiler, which means that the library writer will be required to instantiate every branch of the template just to make sure the syntax is correct. This work is traditionally handled by the compiler. *Static_if* can only tokenize code in template definitions, so templates using the features cannot have their types or semantic errors checked for without a proper AST.

**Conclusion & Closing Gestures**

While the acceptance and ensuing incorporation of *static_if* into the C++ language is still under review and hot debate, we can clearly identify the need for such a functionality while, at the same time, acknowledge that shortcomings are still present and perhaps a closer analysis or revision of the proposition may be necessary. Indeed, the author of the idea is proposing a radical change that, though seemingly miniscule, could make a significant impact on the process of C++ compilation and preprocess methodology.

It is understandable that a more efficient and encompassing means of scoping and handling preprocess predicates is needed. With increasing code complexity as well as increasing per-process data quantity, the importance of expedited and compact running code is further pressing upon us. From the standpoint of a languages designer, though, it is clear that more thought is warranted before this idea is implemented. As Bjarne Stroustrup so accordingly points out, while simplicity is good, too much simplicity may actually result in a juxtaposing abstract result. Rather than cut down features, perhaps additions or fine tuning to current preprocessing semantics would be more beneficial and understandable, as a whole, to the programming community.

Sources
=======

1. Brown, Walter E. "A Preliminary Proposal for a Static If." Thesis. Open Standards, 2011. Open Standards. 11 Dec. 2011. Web. 20 Apr. 2013. <http://open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3322.pdf>.

2. "C Preprocessor." Wikipedia. Wikimedia Foundation, n.d. Web. 20 Apr. 2013. <http://en.wikipedia.org/wiki/C_preprocessor>.

3. McMahon, Kurt. "The C++ Compilation Process." The C++ Compilation Process. Kurt McMahon, n.d. Web. 20 Apr. 2013. <http://faculty.cs.niu.edu/~mcmahon/CS241/Notes/compile.html>.

4. Stroustrup, Bjarne, Gabriel Dos Reis, and Andrew Sutton. "“Static If” Considered." Thesis. Open Standards, 2013. Open Standards. 16 Mar. 2013. Web. 20 Apr. 2013. <http://open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3613.pdf>.
