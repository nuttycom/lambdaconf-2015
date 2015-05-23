## Parametricity: The Essence of Information Hiding

[Kris Nuttycombe](http://github.com/nuttycom) -- [`@nuttycom`](http://twitter.com/nuttycom)

April 19, 2014

<http://github.com/nuttycom/lambdaconf-2015>

------

## About Me

- Introduced to Scala in 2008
- Discovered Scalaz in 2010
- Led Scala dev teams at Gaiam, Precog, & Simple Energy

- Finally started using Haskell for real work in 2014

------

## About This Talk

> - Mostly (very) introductory material
> - Some Haskell basics
>     - Types
>     - Universal Quantification
>     - What Parametricity Means
>     - Existential Interlude
> - How to apply what we learn in Haskell to work we do in other languages

<div class="fragment">

**Interrupt me!**

</div>

<div class="notes">
Because this is an introductory talk, I want it to be interactive.

At this point, ask for show of hands:

# who have experience with a functional programming language
# who have experience with a typed functional programming language
# who have experience with an OO language with parameterized types (generics)

So, what is a type?
</div>

------

## Types

> - A type is a set of possible values
> - e.g. 
>     - the type Bool is `{ True, False }`
>     - the type Int is `{ -2147483648, ..., 2147483647 }`

<div class="fragment">

*Type* is purely a concern of static analysis.

</div>

<div class="notes">

If you write a piece of code, you have to reason about what
values are representable in that piece of code. This is true
about *every* programming language. You simply may not have
access to a program that helps you with that kind of reasoning.

And, in complete fairness, some terms are *hard* to type. Take
any example where some feature of a system determined at runtime
defines the operations that are available with respect to a value. 
ActiveRecord is a good example of this. 

</div>

------

# ∀

~~~{.haskell }

id :: ∀ a. a -> a
  
~~~

- `::` is pronounced "has type"
- `∀`  is pronounced (and may be written) `forall`

<br/>

> - A type is a set of possible values
> - A type *variable* is the set of such sets.

------

<img src="./img/bttf-19890601.png" width="400"/>

> "Write down the definition of a polymorphic function on a piece of paper. Tell me its type, but careful not to let me see the function's definition. I will tell you a theorem that the function satisfies." --[Wadler, 1989](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.38.9875&rep=rep1&type=pdf)

------

## Simple Examples

~~~{.haskell }

asdf :: ∀ a. a -> a -> a
   
~~~

<div class="fragment">

- The output value is one of the input values.
- We can't tell which one, but it will always be the same one.

</div>

------

## Simple Examples

~~~{.haskell }

asdf :: ∀ a b. a -> b -> a
   
~~~

<div class="fragment">

- The value returned is the first argument to this function.
- The second argument is to this function is ignored entirely.

</div>

------

## Simple Examples

~~~{.haskell }

asdf :: ∀ a b. (a, b) -> a
   
~~~

<div class="fragment">

- The value returned is the first element of the tuple.
- The second element of the tuple is ignored entirely.

</div>

<div class="notes">

This is obvious, right? This is the 'first' function, which just uncurries the
function on the former slide, which was 'const'.

</div>

------

> "Write down the definition of a polymorphic function on a piece of paper. Tell me its type, but careful not to let me see the function's definition. I will tell you a theorem that the function satisfies." --[Wadler, 1989](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.38.9875&rep=rep1&type=pdf)

*Note that this only says "I will tell you `a` theorem."*

------

## Simple Examples

~~~{.haskell }
asdf :: ∀ a b. (a -> b) -> [a] -> [b]
~~~

> - Every value in the output is obtained by applying the function provided to a
    member of the input.
> - *BUT* `List` gives the implementer too much information, and the caller too little.

<div class="fragment">
~~~{.haskell }
asdf :: ∀ a b. (a -> b) -> [a] -> [b]
asdf f [] = []
asdf f (x : _) = [f x]
~~~
</div>

<div class="notes">
- could reverse the result
- could just pick the n'th value from the list, apply the function, and
  repeat it indefinitely...
</div>

------

<img src="./img/list_fns.png" width="800"/>

(and many more)

------

## Code Reuse

~~~{.haskell }

asdf :: ∀ a b. (a -> b) -> [a] -> [b]

firsts :: ∀ a b. [(a, b)] -> [a]
firsts xs = asdf fst xs
-- firsts = asdf fst 
  
~~~

- This is excessively restrictive for the caller, who must provide a list
- AND excessively permissive to the implementer, who could (ab)use other List functions!

<div class="notes">

This is just a small motivating example.

If `List` gives us too much information, how can we recover generality?
To start with, remember that 

</div>

------

## Typeclasses

~~~{.haskell }

class Functor f where
  fmap :: ∀ a b. (a -> b) -> f a -> f b
  
instance Functor List where
  fmap :: ∀ a b. (a -> b) -> [a] -> [b]
  fmap f [] = []
  fmap f x : xs = f x : fmap f xs
  
~~~

------

~~~{.haskell }

firsts :: ∀ f a b. Functor f => f (a, b) -> f a
firsts = fmap fst 
  
~~~

------


~~~{.haskell }

class Foldable f where
  foldr :: ∀ a b. (a -> b -> b) -> b -> f a -> b

class Monoid a where
  mempty :: a
  mzero :: a -> a -> a
  
~~~

<div class="fragment">

~~~{.haskell }

foldMap :: 
  ∀ f m a. (Foldable f, Monoid m) => 
  (a -> m) -> f a -> m
  
~~~

</div>

------

## Laws
~~~{.haskell }

class Functor f where
  -- | Functor laws! 
  --
  -- >>> fmap id == id
  -- >>> fmap (f . g) == fmap f . fmap g 
  --
  fmap :: ∀ a b. (a -> b) -> f a -> f b
  
~~~

- This would be a nice and complete solution ... but it's not something Haskell does.

------

- However, we can get a little bit of the way there.

~~~{.haskell }

-- | reverse a list
--
-- >>> import Test.DocTest.Prop
-- >>> prop $ \xs ys ->
--       rev (xs ++ ys) == rev ys ++ rev xs
--
-- >>> prop $ \xs -> rev (rev xs) == xs
--
rev :: ∀ a. [a] -> [a]
  
~~~

------

## Pitfalls

~~~{.haskell }

asdf :: ∀ a. a
  
~~~

<div class="fragment">

- This function will cause your program to fail at runtime.
- If we have this in our language, we can't trust anything.

</div>

<div class="fragment">

<br/>

This literally says, "all propositions are true."

</div>


<div class="notes">

This is a really interesting result. From nothing but the type
signature, we can tell right away that something is wrong. This
function promises that, for any type, it can return a value of that
type. Sorcery! This is the function that could put all of us
out of our jobs instantaneously, if it were to exist. It promises
to be able to synthesize literally any program from thin air.

</div>

------

<img src="./img/discordia.png" width="800"/>

> -- [The Principia Discordia](http://www.principiadiscordia.com/downloads/Principia%20Discordia.pdf)

------

## Give Up Now?

~~~{.haskell }

error :: ∀ a. String -> a
  
~~~

------

### Fast And Loose Reasoning Is Morally Correct

> "Functional programmers often reason about programs as if they were written in a total language, expecting the results to carry over to non-total (partial) languages. We justify such reasoning." -- [Danielsson et al. 2006](http://www.cse.chalmers.se/~nad/publications/danielsson-et-al-popl2006.html)

<div class="notes">

It is proved that if two closed terms have the same
semantics in the total language, then they have related
semantics in the partial language. It is also shown that the
PER gives rise to a bicartesian closed category which can be
used to reason about values in the domain of the relation.

</div>

------

## Seeing The Fnords

~~~{.haskell }
brokenHead :: ∀ a. [a] -> a
~~~

<div class="fragment">
~~~{.haskell }
fromJustAwful :: ∀ a. Maybe a -> a
~~~
</div>

<div class="fragment">
~~~{.haskell }
stupIdx :: ∀ a. [a] -> Int -> a
~~~
</div>

<div class="fragment">
~~~{.haskell }
horkl1 :: ∀ a. (a -> a -> a) -> [a] -> a
~~~
</div>

------

## Universal Problem

~~~{.haskell }
data JsonRendering a = 
  JsonRendering a (a -> Data.Aeson.Value)

jsonArray :: 
  ∀ f a. (Foldable f) => 
  f (JsonRendering a) -> Data.Aeson.Value
~~~

------

## Existential Solution

- This is something you don't need very often, but when you do, it's essential.

~~~{.haskell }
{-# LANGUAGE ExistentialQuantification #-}

data JsonRendering = 
  ∀ a. JsonRendering a (a -> Data.Aeson.Value)

jsonArray :: 
  ∀ f. (Foldable f) => 
  f JsonRendering -> Data.Aeson.Value
~~~

------

## Bringing Parametricity Home

------

~~~{.haskell}
const :: ∀ a b. a -> b -> a
const a _ = a
~~~

~~~{.java}
static <A, B> Function<B, A> constf(A a) {
  return (B b) -> a;
}
~~~

~~~{.ruby}
def constf(a)
  lambda do |b|
    a
  end
end
~~~

------

~~~{.scala}

trait Monoid[A] {
  def mempty: A
  def mappend(a0: A, a1: A): A
}
  
~~~

~~~{.scala}

trait Functor[F[_]] {
  def fmap[A, B](f: A => B)(fa: F[A]): F[B]
}
  
~~~

------

~~~{.java}
// this works
interface Monoid<A> {
  A mempty();
  A mappend(A a0, A a1);
}
~~~

<div class="fragment">
~~~{.java}
// This doesn't work. At all.
interface Functor<A> {
  <B> Functor<B> fmap(Function<A, B>);
}

class MyList<A> extends Functor<A> {
  public <B> MyList<B> fmap(Function<A, B>);
}
~~~
</div>

------

~~~{.java}

List<String> result =  
  Arrays.asList("Larry", "Moe", "Curly")
        .stream()
        .map(s -> "Hello " + s)
        .collect(Collectors.toList());

// result will be a List<String> containing 
// "Hello Larry", "Hello Moe" and "Hello Curly"
     
~~~

<div class="fragment">

~~~{.java}

static <T> Collector<T,?,List<T>> toList()
   
~~~

</div>

------

<img src="./img/quack.jpeg" width="700"/>

------

## A Few Quick Rules

> - Give the consumer of a function the most freedom, and the implementer the least.
> - Never use introspection to determine the types of your arguments (type erasure is a GOOD thing!)
> - Prefer typeclasses over inheritance if your language will let you.

<div class="notes">
  One of the nice things about dynamically typed languages is that you don't 
  suffer the "MyType" curse. This may explain their popularity over things 
  like Java.
</div>

------

# Questions?
