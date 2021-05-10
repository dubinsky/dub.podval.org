---
layout: page
math: true
---
# Lambda-calculus #

## Definition ##

When we get to dependent types, where well-formedness of the terms depends on the
types of the variables, terms will need to be defined using judgements; here a
simple grammar is sufficient; assuming infinite countable set Variable of variables:

Term ::=          (t, u...)
 - Variable |     (x, y, ...)
 - λ Variable . Term |  (_abstraction_)
 - Term Term            (_application_)

Conventions:
  - application associates to the left: tuv means (tu)v;
  - application binds stronger than the abstraction: λx.xy means λx.(xy);
  - abstractions can be grouped: λxyz.xz(yz) means λx.λy.λz.xz(yz).

Variable x in λx.t is _bound_; variable than is not bound is _free_;
term with no free variables is _closed_.

_χ-reduction_ is the smallest binary relation on terms
such that when t →<sub>χ</sub> t' then also:
- λx.t →<sub>χ</sub> λx.t';
- tu →<sub>χ</sub> t'u;
- ut →<sub>χ</sub> ut'

and:

α-reduction: λx.t →<sub>α</sub> λy.t', where t' is t with all free occurrences of x renamed to y.

β-reduction: (λx.t)u →<sub>β</sub> t[u/x], where substitution t[u/x] is the t
with all free occurrences of x replaced by u ((λx.t)u is called a _redex_);

η-reduction (extensionality): λx.t →<sub>η</sub> t.

When substituting a term for a bound variable, care needs to be taken not to
accidentally "capture" a variable that is free in that term. This is a great pain
for the implementer, and various tricks were developed to ease it (de-Brujin
indices, Barendregt convention); we will say no more about it, and just assume
that before substitution all variables that need to be distinct are given fresh names :)

_χ-expansion_ is a relation opposite to →<sub>χ</sub>. 

→*<sub>χ</sub> (multi-step reduction; reduction path) is the reflexive and transitive
closure of →<sub>χ</sub>.

=<sub>χ</sub> (χ-equivalence, χ-convertability) is the symmetric closure
of →*<sub>χ</sub>.

Term that can not be reduced is in _normal form_ (a _value_).

Term t is _weekly normalizing_ if there exist a normal form u such that t →*<sub>β</sub> u.

Term is _strongly normalizing_ when every sequence of reductions will eventually produce a
normal form.

Not all terms a strongly normalizing: Ω = (λx.xx)(λx.xx) reduces
to itself. But, if t →*<sub>β</sub> u<sub>1</sub> and t →*<sub>β</sub> u<sub>2</sub>,
there exists such v that u<sub>1</sub> →*<sub>β</sub> v and u<sub>2</sub> →*<sub>β</sub> v
(_confluence_, _Church-Rosser property_).


## Encodings ##

It is possible to encode common types and data structures in lambda calculus.

identity: I = λx.x

boolean 'true': T = λxy.x

boolean 'false': F = λxy.y

if-then-else: if = λbxy.bxy (if T t u →*<sub>β</sub> t etc.)

logical operations: and = λxy.xyF; or = λxy.xTy; nor = λx.xFT;

product: pair = λxyb. if b x y 

projections: fst = λp.pT; snd = λp.pF (fst(pair t u) →*<sub>β</sub> t etc.)

(n-tuples can also be defined)

n-th Church numeral: **n** = λfx.f(f(...(fx))) where f is applied n times

0 = λnfx.x

succ = λnfx.f(nfx)

add = λmnfx.m succ n

mul = λmnfx.m (add n) 0

exp = λmn.n (mul m) 1

iszero = λnxy.n(λz.y)x

pred = λnfx.n(λgh.h(gf))(λy.x)(λy.y)

sub = λmn.n pred m

leq = λmn.iszero (sub m n)

## Fixpoints ##

u is a _fixpoint_ of term u if t u →*<sub>β</sub> u; in lambda-calculus,
**every** term has a fixpoint, **and** there is a term Y (_fixpoint combinator_)
such that Yt is a fixpoint of t! For example:
- _Curry fixpoint combinator_: Y = λf.(λx.f(xx))(λx.f(xx))
- _Turing fixpoint combinator_: ϴ = (λfx.x(ffx))(λfx.x(ffx))

(Fixpoint combinator is the Russel paradox in disguise when terms are read as
predicates, λx.t is read as {x | t}, and tu is read as u ∈ t.)

Functions definable in lambda-calculus are precisely the recursive ones 
(_Kleene theorem_).

Instead of encoding things like products, new term forms can be added
together with the corresponding reduction rules.

## Reduction strategies ##

Orders on redexes:
 - all the redexes of t and u are _inside_ the redex (λx.t)u;
 - all the redexes of t are to the _left_ of all the redexes of u in tu.

Reduction strategy determines the order in which the redexes in a term get reduced:
- _innermost_ (=call by value) (_outermost_ (=call by name)) strategy selects the most _inside_ (_outside_) redexes;
- _left_ (_right_) strategy selects the the most _left_ (_right_) redexes.
- _week_ strategy never reduces abstractions λx.t, even if there are redexes in t;
- left strategy is _head_ if it never reduces variable applications x t1 ... tn even
  if some ti has redexes.
- _normal order_ strategy is leftmost outermost; it is normalizing: if the term has
  a normal form, this strategy will find it (_standartization theorem_).

## Combinatory logic ##

All λ-terms can be built (up to β-equivalence) from just three _combinators_, thus avoiding
variables, α-conversion and all that, and formulating β-reduction directly on the combinators:
- I = λx.x (identity; using the variable);
- S = λxyz.(xz)(yz) (duplicating the variable);
- K = λxy.x (erasing a variable).

Actually, just S and K suffice as the "basis" of the λ-calculus, 
since I can be defined as S K K.

Indeed, one combinator is sufficient: ι = λx.xSK; we can then define:
- I = ιι;
- K = ι(ι(ιι));
- S = ι(ι(ι(ιι))).

So, every λ-term can be encoded as a binary word ;)

Besides being cute, nameless (combinator-based) representations of λ-terms are
used in implementation of functional programming languages.

## Bibliography ##

[Mim20] "Program = Proof", Mimram, [2020](https://www.lix.polytechnique.fr/Labo/Samuel.Mimram/teaching/INF551/course.pdf)

[SU06] "Lectures on the Curry-Howard Isomorphism", Sorensen & Urzyczyn, 2006.

[Sel13] "Lecture notes on the lambda calculus", Selinger, [2013](https://arxiv.org/pdf/0804.3434.pdf).

[Bar84] "The Lambda Calculus: Its Syntax and Semantics", Barendregd, 1984.

[NG14] "Type Theory and Formal Proof", Nederpelt & Geuvers, 2014

[Sch24] "Uber die Bausteine der mathematischen Logik", Schonfinkel, 1924.