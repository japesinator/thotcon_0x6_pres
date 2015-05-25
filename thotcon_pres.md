Curry and TARTS
===============

JP Smith, THOTCON 2015
----------------------

@japesinator, github/japesinator
--------------------------------

---

The Big Problem
---------------

  * Reasoning about programs is hard

  * Making guarantees about programs is hard

  * What runs in your head and what runs on the machine can be pretty different

---

How it Bites Us
---------------

  * In way too many ways to cover in a fast talk

  * One good example: side-channel attacks

  * Breaking crypto by looking at things other than math (time to execute, power
    draw, etc.)

---

How Timing Attacks Work
-----------------------

  * Consider equality

  * We want to compare two bytes

----

Bad Program
-----------

```
function eq(byte0, byte1):
  for i in range(0,7):
    if byte0[i] != byte1[i]:
      return False
  return True
```

  * Problem: it takes less time if a byte is different at the start than at the
    end! (`00000000` vs `00000001`: 8 loops, `00000000` vs `10000000`: 1 loop)

  * We can guess if one byte is equal to another by doing a binary search

  * If we use this for passwords (please don't ever do this), attacker can just
    guess passwords and see what takes longer!

----

Other examples
--------------

  * That time they broke RSA (Kocher in 1999)

  * That time they broke SSL (Boneh and Brumley in 2003)

  * That time they broke AES (Boneh in 2005)

---

So What Can We Do?
------------------

  * Code review

  * Unit tests

  * Property based testing

  * Fuzzing

  * All nice, but how can we get an ironclad guarantee?

---

And Now For Something Completely Different
------------------------------------------

---

Type systems
------------

  * `5 :: Int` says 5 is an integer

  * `f :: Int -> String` says f is a function from integers to strings

  * `f x = x + 3` won't compile because it's actually a function from integers
    to integers

----

The Curry-Howard isomorphism
----------------------------

  * Types == Propositions

  * Things with those types == Proofs

  * Code compiling == Those proofs are valid!

----

Example
-------

```
length :: String -> Int
-- If there exist strings, there exist integers
length [] = 0
length (x :: xs) = 1 + length xs
-- Here's why
```

----

But That's Not All!
-------------------

  * Dependent types let us have functions from whatever we want to types

  * For example:

```
superComplex : String -> Int -> (Int -> [Int]) -> Type
```

  * These exist right now!

----

  * In languages like Idris, Agda, and Coq, we can write whatever propositions
    we want as types

  * Then, if our code compiles, we've proved those propositions

---

Putting It All Together
-----------------------

  * We can reason about our function using types!

  * We can mathematically prove properties like resistance to timing attacks
    so that our code can't compile if it's vulnerable to timing attacks

---

Demo!
-----

---

Example Code
------------

* First, we write a function that checks equality

```
-- Check if two bytes are equal
byteEq : Byte -> Byte -> Bit
byteEq = zipAndFold bitNXor bitAnd

```

----

* Then, we modify it to keep track of how long it takes

```
-- Check if two bytes are equal and keep track of how
-- long it takes
countingByteEq : Byte -> Byte -> (Bit, Nat)
countingByteEq a b = zipAndFold (addCount bitNXor)
                                (addCount bitAnd)
                                (initializeCount a)
                                (initializeCount b)

```

----

* Then, we can prove it will always take the same amount of time!

```
-- Proof that an implementation of equality takes
-- the same time for any two bytes
timeConstancyOfEq : (a, b, c, d : Byte) ->
  time $ countingByteEq a b = time $ countingByteEq c d
timeConstancyOfEq a b c d =
  rewrite numericTimeConstancyOfEq a b in
  rewrite numericTimeConstancyOfEq c d in
  Refl
-- Working code for the whole thing at goo.gl/7E4n85
```

---

More Demo!
----------

---

Disclaimers
-----------

  * Right now, this works mostly only in super-impractical academia land

  * Please do not write any production code in Idris/Agda/Coq, Bad Things will happen

---

Questions?
----------
