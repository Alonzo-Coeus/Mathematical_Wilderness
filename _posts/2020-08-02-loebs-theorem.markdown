---
layout: post
title:  "Loeb's Theorem in Haskell"
date:   2020-08-02 19:04:38 +0100
categories: haskell logic godel
---
# Table of Contents

1.  [What is Loeb&rsquo;s theorem and what are it&rsquo;s implications?](#org55bc79c)
2.  [Loeb&rsquo;s theorem in Haskell](#org681f198)
3.  [Playing with Loeb&rsquo;s theorem](#org90a76af)
4.  [Conclusion](#org60071fd)
5.  [Additional Sources](#org6dc761b)



<a id="org55bc79c"></a>

# What is Loeb&rsquo;s theorem and what are it&rsquo;s implications?

Loeb&rsquo;s theorem is a re-implementation of godel&rsquo;s theorem within modal logic.
Allowing us to ignore &ldquo;non-essential&rdquo; implementation details such as godel numbering and how programs are actually encoded.
Roughly Loeb&rsquo;s theorem states that if it is provable that a proof of a fact x implies x is true then it is provable x is true.
Or in the language of modal logic ☐(☐x -> x) -> ☐x

If the statement x in Loeb&rsquo;s theorem is taken to be &perp; loeb&rsquo;s theorem simplifies as follow

x = &perp;  
☐(☐ &perp; -> &perp;) -> ☐ &perp;  
☐(¬ ☐ &perp;) -> &perp;  
¬ ☐ (¬ ☐ &perp; )  

Which translates in english is equivalent to saying &ldquo;it is not provable that false-hood is not provable&rdquo; or in other words &ldquo;A logical system can&rsquo;t assurt it&rsquo;s own consistency&rdquo;.
Which is one of the primary insights offered by Godel&rsquo;s theorem.

Other implications of Loeb&rsquo;s theorem is that of the &ldquo;loebstical&rdquo; discussed in large-part by MIRI and lesswrong.
In short the &ldquo;loebstical&rdquo; is the problem faced by self-improving AI programs, where they cannot prove facts about future agents without them being of a weaker axiomatic system.
This damages the possibilty of creating a recursive self-improving AI as for every rewrite it would have to downgrade to a less powerful axiomatic system.
As well as this the loebstical also makes it difficult for AI systems to reason about equally powerful systems or work within enviroment more powerful than themselves, which is a requirement for AGI as it would be embedded within it&rsquo;s enviroment, and thus cannot be more powerful than it.


<a id="org681f198"></a>

# Loeb&rsquo;s theorem in Haskell

First let&rsquo;s describe modal logic.
For a proof of lob&rsquo;s theorem we need three axioms, distribution, necessitiation, and internal-necessitiation (which we will ignore as it&rsquo;s a special case of necessitation)

    class Modality m where  -- define a modal logic
      distribution :: m (a -> b) -> (m a -> m b)
      necessitation :: a -> m a -- if it's true it's provable

To prove loeb&rsquo;s theorem within this defenition of Modal logic, we will need construct a function of type.

    loeb :: Modality m => m (m a -> a) -> m a

When the type constructor used is &ldquo;quotation&rdquo; this becomes a programatic implementation of loeb&rsquo;s theorem as the modality can be viewed as an applicitive wrapping a type implemented within a DSLs AST.
As Curry-Howard shows an isomorphism between the AST type-constructor in a language (lispers think quote) and the provibality modality within logic.

However before we can implement loeb&rsquo;s theroem we should first implement fmap with the provided methods given by the modality type-class.

    modal_fmap :: Modality m => (a -> b) -> (m a -> m b)
    modal_fmap = distribution . necessitiation

The logic behind this is fairly simple with us first wrapping the input function in the modality and then distributing said modality.
This achieves the fmap type signature as well as well as following the composition rules we all know and love.

Now using modal<sub>fmap</sub> we can implement loeb&rsquo;s theorem.

    loeb = fix (modal_fmap . flip id =<<)

This implementation is rather elagant in my eyes, as it&rsquo;s point free and shows loeb&rsquo;s theorem as a modfied fixed point combinator.
However the use of a fixed point combinator also makes it an invalid proof as the implementation does not posses totality.


<a id="org90a76af"></a>

# Playing with Loeb&rsquo;s theorem

One of the interesting things about loeb is it&rsquo;s similarity with the fixed point combinator&rsquo;s type.

    fix  ::  (a -> a) -> a
    loeb :: Modality m => m (m a -> a) -> m a

If you drop the modality, for example by using the identity Applictive both become equivalent.
The deeper reason for this rather than just dropping modality terms is by using the identity applicitive as your notion of provability it becomes isomorphic to truth within the system.
However the cost of doing this is that your logic becomes inconsitent as constructing a value of any type becomes trivial as the fixed point of the identity function is a member of all types.
Thus making all statements trivialy true, as a side effect of attempting to make a logic both complete (a -> Prov a) and consistent (Prov a -> a).
Which also means that self-interpreters are impossible in any theorem proving system as it would lead to all statements being trivialy true which detracts from the usefulness of such a system.

Another interesting note about loeb is that if you for example use a List as the &ldquo;modality&rdquo; the loeb&rsquo;s theorem becomes a spreasheet evaluator (This isn&rsquo;t my discovery &ldquo;A neighborhood of infinity&rdquo; first posted it to my knowledge).

    instance Modality [] where
      distribution = (<*>)
      necessitation = pure
    
    main = print $ loeb [(flip (!!)) 1, pure 2, ((+) 1 . ((flip (!!)) 0))]

Returns:

    [2,2,3]


<a id="org60071fd"></a>

# Conclusion

Loeb&rsquo;s theorem is in my eyes one of the most elagent theorems in logic and computer science, rivaling Curry-Howard and the Church-Turing Thesis.
With it&rsquo;s implications spanning from the improvability of improvability of false-hood, to the impossibility of self-interpeters as they&rsquo;d lead to inconsistent logics.
But I think the fact that one of the most profound results of 20th century meta-mathematics is effectively a spreadsheet interpreter makes Loeb&rsquo;s theorem by far the funniest theorem in meta-mathmatics and logic.


<a id="org6dc761b"></a>

# Additional Sources

-   [ncat-lab](https://ncatlab.org/nlab/show/L%C3%B6b%27s+theorem)
-   [A neighborhood of infinity (who I think first noted the spreasheet thing)](http://blog.sigfpe.com/2006/11/from-l-theorem-to-spreadsheet.html)

