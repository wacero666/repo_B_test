Here are a few notes to help understanding of the sample solution.

First, the line 
```java
  results.put(pool.submit(() -> candidate.isProbablePrime(50) ? candidate : null));
```
is a key line of code for this sample solution.  It's concisely written as a single line 
(this is how I would write it in a production system!) but it can be clearer if written
as multple lines.  It's equivalent to:
```java
  Callable<BigInteger> primalityCheckingTask = new Callable<BigInteger> {
      @Override
      BigInteger call() {
          if (candidate.isProbablePrime(50)) { 
              return candidate;
          } else {
              return null;
          }
      }
  };
  
  Future<BigInteger> primalityCheckResult = pool.submit(primalityCheckingTask);
  results.put(primalityCheckResult);
```

In other words, the single line of code is creating a new Callable object that returns the
candidate Mersenne number if it is prime, and null otherwise.  It's also worth noting that 
due to the way that scope works for inner classes in Java, the `Callable`'s `BigInteger` `candidate`
is unchanging (even though the variable `candidate` takes on different values for different
executions of the loop).

Second, it's worth noting that the design of this program consists of multiple implementations
of the producer-consumer pattern: there's one explicit use of this pattern and one implicit use
of this pattern.  For the explicit use of the pattern, the line
```java
  new Thread(() -> generateCandidates(pool, results)).start();
```
starts a single producer thread that generates the results of primality-checking candidate Mersenne
numbers sequentially in increasing order.  The main thread continues as a single-threaded consumer, 
processing the results of the primality checks (ignoring non-prime candidates, and counting and
printing the Mersenne candidate if it was prime).  Because there's one sequential producer thread
and one sequential consumer thread and results are inserted into the `results` queue in increasing
order, the program guarantees that Mersenne primes will be printed in increasing order.

There's also an implicit use of the producer consumer pattern via the use of the `ExecutorService`,
`pool`.  Here the `generateCandidates` function is again a single-threaded producer, submitting
primality-checking tasks to the `ExecutorService`.  The `ExecutorService` is implicitly maintaining
a queue (in arbitrary order) and managing a set of `N_THREADS` consumer threads, which run the 
primality-check code and generate the `Future<BigInteger>` results.
