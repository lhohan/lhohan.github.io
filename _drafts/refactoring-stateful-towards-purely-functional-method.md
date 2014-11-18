Refactoring a side effecting method to a purely functional one
============

Currently I am working my way to Functional Programming in Scala [TODO link] and thus far I really like it.
Mind you, this book is not for the faint of heart, it contains **load** of exercises of which one series I wanted to get to the gist of it, and explain here.
 It concerns refactoring a side effecting method to a purely functional one using a mechanism that can be applied over and over again.

The example: rolling a die with a flawed implementation.


 We start with following implementation of `rollDie`:

 ```scala
import scala.util.Random

object DieRoller {
  def rollDie: Int = {
    val rng = Random
    rng.nextInt(6)
  }
}
```

The above contains an error as Random is zero based. So may occasionally get zero as the result of our die roll.

A test to expose this behaviour:

```scala
import org.scalatest.FunSuite

class DieRollerTest extends FunSuite {
  import DieRoller._

  test("roll should be between in range 1 to 6") {
    val _3Rolls = (1 to 3).map(x => rollDie)
    _3Rolls.foreach { roll =>
      assert(roll > 0 && roll <= 6, s"error: $roll")
    }
  }
}
```

Basically we assert that roll of the die should be between one of 1 to 6.

This test will fail randomly (I check 3 rolls at once to make it fail more often).

As it is now, the fact that we use `Random` inside our implementation of `rollDie`, makes it basically impossible to
consistently test and reproduce our error.

So how do we make our method referentially transparent?

The key is to make state updates *explicit* by returning the new state along with the value that we are generating! Here is how this looks:

```scala
// The key to recovering referential transparency: make state updates explicit.
// Return the new state along with the value that we are generating

trait RNG {
  def nextInt: (Int, RNG)
}

// Implementation of RNG. Implementation omitted as it is a detail in what we are trying to achieve.
// Detail in the book and in working sample included at the end.

case class Simple(seed: Long) extends RNG {
  def nextInt: (Int, RNG) = {
    // implementation omitted
    (n, nextRNG) // The return value is a tuple containing both  random integer and the next `RNG` state.
  }
}

// keeping it simple, book's impl. is better and more correct but also more complex
def nonNegativeIntLessThan(n: Int)(rng: RNG): (Int, RNG) = {
  val (i, r) = rng.nextInt
  ((if (i < 0) -i else i) % n, r)
}

def rollDie: RNG => (Int, RNG) = nonNegativeIntLessThan(6)

```

If the function type is confusing, alternatively and exactly the same (but less elegant):

```scala
def rollDie(rng: RNG): (Int, RNG) = {
  val (i, r) = rng.nextInt
  ((if (i < 0) -i else i) % 6, r)
}
```







