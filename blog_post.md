# Benchmarking Fibonacci with Go

Go has a really nice testing suite built into the compiler tool chains. Run `go test` in any package directory and it runs all your tests defined in all `*_test.go` files. We can also run benchmarks with during testing by, with `go test -bench=.`. I decided to explore Go's benchmarking tools, and flex some of my algorithm skills in the process. Naturally we implement Fibonacci many time over.

The repository lies at <https://github.com/TerrenceHo/fib>, where you can use git to clone it.  You can also use `go get github.com/TerrenceHo/fib` if you have a go runtime, which you'll need to run tests.  The file `fib.go` holds various fibconacci implementations and some documentation.  The associated tests and benchmarks lie in `fib_test.go`. 

### FibRecursive: Exponential Recursive

```
func FibRecursive(n int) int {
    if n < 2 {
        return n
    }
    return FibRecursive(n-1) + FibRecursive(n-2)
}
```
The most basic Fib sequence algorithm known to man, `FibRecursive` boasts a mighty exponential O(2^n) run time, and is so slow I can't even run `FibRecursive(64)` on my laptop without having to wait for an eternity.  The reason being, for an input _n_, _n+1_ takes almost twice as long to computer, since the recursive tree makes two recursive calls each level and doubles the computation need for the next level.  In our testing suite, we max our runtime calls at _n_=32.  

__Draw Picture of Exponential Graph HERE__

Memory-wise, `FibRecursive` is also inefficient, since each function call creates adds to the function call stack, and so runs the danger of actually running out of memory at high inputs.

### FibRecursiveCache: Exponential Recursive Cache
```
func FibRecursiveCache(n int) int {
    cache := make([]int, n+1, n+1)
    fibRecursiveCache(n, &cache)
    return cache[n]
}

func fibRecursiveCache(n int, cache *[]int) {
    if n < 2 {
        (*cache)[0] = 0
        (*cache)[1] = 1
        return
    }
    fibRecursiveCache(n-1, cache)

    (*cache)[n] = (*cache)[n-1] + (*cache)[n-2]
}
```

We can optimize our previous algorithm by caching previous computed values. In `FibRecursive`, each call ends up recursively recomputing the same values.  For example, `FibRecursive(4)` computes both `FibRecursive(3)` and `FibRecursive(2)`.  However, `FibRecursive(3)` will also computer `FibRecursive(2)`, an unnecessary computation.  Thus, we can cache previous calculated Fibonacci values in an array.  In doing so, we lower our runtime from exponential to linear.

`FibRecursiveCache` recurses from _n_ to 1, and then builds the cache going backwards.  The left-most value in our array holds our final answer.  `FibRecursiveCache` ends up with both a linear runtime and linear memory usage, due to the array.  Run `go test -bench=.`, and you can see that `FibRecursiveCache` is much faster, even on lower inputs.  (Remember for benchmarks, lower _ns/op_ is better.)

__INSERT GRAPHS HERE, CROSSED OUT__

### FibIterative: Linear Iterative Implementation
```
func FibIterative(n int) int {
    var temp int
    first := 0
    second := 1
    for i := 0; i < n-1; i++ {
        temp = second
        second = first + second
        first = temp
    }
    return second
}
```

`FibIterative` takes the concept that we do not really need to keep every previous computed value, only the previous number. Thus we get rid of the cache entirely and instead add the two previous numbers together in a loop. This eliminates the need to keep memory, while keeping the runtime linear.  `FibIterative` is faster on benchmarks than the other iterations, due to the low overhead of for loops.

### FibTailRecursive: Tail Recursive Implementation

```
func FibTailRecursive(n int) int {
    return fibTailRecursive(n, 0, 1)
}

func fibTailRecursive(n, first, second int) int {
    if n == 0 {
        return first
    }
   
    return fibTailRecursive(n-1, second, first+second)
}
```
`FibTailRecursive` does not build a cache, but is really a recursive implementation of the iterative implementation. This is more akin to a "functional" implementation, replacing the for loops, a construct that does not exist in most functional languages.  If you consider recursive calls taking memory on the call stack, this is linear in both runtime and memory.  

### FibPowerMatrix: Linear Matrix Implementation
```
func FibPowerMatrix(n int) int {
    F := [2][2]int{
        [2]int{1, 1},
        [2]int{1, 0},
    }
    if n == 0 {
        return 0
    }
    fibPower(&F, n-1)
    return F[0][0]
}

func fibPower(F *[2][2]int, n int) {
    M := [2][2]int{
        [2]int{1, 1},
        [2]int{1, 0},
    }
    for i := 2; i <= n; i++ {
        fibMultiply(F, &M)
    }
}

func fibMultiply(F *[2][2]int, M *[2][2]int) {
    f := *F
    m := *M
    x := f[0][0]*m[0][0] + f[0][1]*m[1][0]
    y := f[0][0]*m[0][1] + f[0][1]*m[1][1]
    z := f[1][0]*m[0][0] + f[1][1]*m[1][0]
    w := f[1][0]*m[0][1] + f[1][1]*m[1][1]

    (*F)[0][0] = x
    (*F)[0][1] = y
    (*F)[1][0] = z
    (*F)[1][1] = w
}
```
Thus far, we've gone from exponential to linear, but there is still a faster runtime achievable. To see how we can achieve a theoretical faster runtime, we first implement a linear matrix multiplication Fibonacci solver.  __INSERT EXPLANATION HERE__

### FibPowerMatrixRecursive: Log(n) Matrix Implementation
```
func FibPowerMatrixRecursive(n int) int {
    F := [2][2]int{
        [2]int{1, 1},
        [2]int{1, 0},
    }

    if n == 0 {
        return 0
    }
    fibPowerRecursive(&F, n-1)
    return F[0][0]
}

func fibPowerRecursive(F *[2][2]int, n int) {
    if n == 0 || n == 1 {
        return
    }

    M := [2][2]int{
        [2]int{1, 1},
        [2]int{1, 0},
    }
    fibPowerRecursive(F, n/2)
    fibMultiply(F, F)
    if n%2 != 0 {
        fibMultiply(F, &M)
    }
}
```
We proved previously that matrix multiplication can help us solve the Fibonacci sequence. However, it was still a linear time implementation. With a little algorithms trick, we can bring the complexity time down to _logn_.  

Ignoring matrices, take for example 2^8.  We can compute that as 2 * 2 * 2 * 2 * 2 * 2 * 2 * 2, and multiple across, which would require 7 multiplication operations. Order of operations does not matter, so we could multiply 2^4, and then square the result to achieve 2^8.  We can recursively apply the same trick to 2^4.  Essentially, on each level we do half the work. This process saves us 4 multiplication operations, compared to the 7 if we had simply multiplied across.  This process is a form of divide and conquer.

We can apply this same process to the matrix multiplication when calculating Fibonacci. `fibPowerRecursive` makes recursive calls to itself, then squares the result.  If _n_ was odd, then we multiply by _M_ one more time.  On each recursive level, we halve the work, so complexity becomes logn. 

When running the benchmarks, we still see that the iterative implementation is faster, due to the overhead involved in matrix multiplication operations.  Theoretically, `FibPowerMatrixRecursive` should be faster on very large Fibonacci inputs, as the overhead involved in matrix multiplication becomes less compared to the operations the `FibIterative` must do.

__INSERT EXPLANATION ABOUT DIVIDE AND CONQUER HERE, WITH EXPONENTIALS__

### Benchmarks
We test our benchmarks on values Fib(n), where n = {1,2,4,8,16,32,64,128,1024} (for `FibTestRecursive`, we stop at n = 32 because any higher values of n simply takes too long to run). 

Here are the benchmarks on for `fib` on my computer.

```
BenchmarkFibRecursive1-8                        2000000000               1.93 ns/op
BenchmarkFibRecursive2-8                        300000000                5.31 ns/op
BenchmarkFibRecursive4-8                        100000000               16.9 ns/op
BenchmarkFibRecursive8-8                        10000000               128 ns/op
BenchmarkFibRecursive16-8                         200000              6196 ns/op
BenchmarkFibRecursive32-8                            100          13737437 ns/op
BenchmarkFibIterative1-8                        1000000000               2.09 ns/op
BenchmarkFibIterative2-8                        500000000                3.03 ns/op
BenchmarkFibIterative4-8                        300000000                4.36 ns/op
BenchmarkFibIterative8-8                        200000000                6.68 ns/op
BenchmarkFibIterative16-8                       100000000               11.4 ns/op
BenchmarkFibIterative32-8                       50000000                26.5 ns/op
BenchmarkFibIterative64-8                       50000000                40.7 ns/op
BenchmarkFibIterative128-8                      20000000                84.1 ns/op
BenchmarkFibIterative1024-8                      2000000               621 ns/op     <--- HERE
BenchmarkFibRecursiveCache1-8                   50000000                25.2 ns/op
BenchmarkFibRecursiveCache2-8                   50000000                30.6 ns/op
BenchmarkFibRecursiveCache4-8                   50000000                38.9 ns/op
BenchmarkFibRecursiveCache8-8                   30000000                57.1 ns/op
BenchmarkFibRecursiveCache16-8                  20000000                93.2 ns/op
BenchmarkFibRecursiveCache32-8                  10000000               170 ns/op
BenchmarkFibRecursiveCache64-8                   5000000               328 ns/op
BenchmarkFibRecursiveCache128-8                  2000000               633 ns/op
BenchmarkFibRecursiveCache1024-8                  300000              4767 ns/op    
BenchmarkFibTailRecursive1-8                    200000000                6.18 ns/op
BenchmarkFibTailRecursive2-8                    200000000                8.30 ns/op
BenchmarkFibTailRecursive4-8                    100000000               12.7 ns/op
BenchmarkFibTailRecursive8-8                    50000000                24.3 ns/op
BenchmarkFibTailRecursive16-8                   30000000                47.0 ns/op
BenchmarkFibTailRecursive32-8                   20000000                86.8 ns/op
BenchmarkFibTailRecursive64-8                   10000000               183 ns/op
BenchmarkFibTailRecursive128-8                   5000000               342 ns/op
BenchmarkFibTailRecursive1024-8                   500000              2731 ns/op
BenchmarkFibPowerMatrix1-8                      200000000                6.25 ns/op
BenchmarkFibPowerMatrix2-8                      200000000                6.34 ns/op
BenchmarkFibPowerMatrix4-8                      50000000                24.7 ns/op
BenchmarkFibPowerMatrix8-8                      30000000                57.8 ns/op
BenchmarkFibPowerMatrix16-8                     10000000               123 ns/op
BenchmarkFibPowerMatrix32-8                      5000000               255 ns/op
BenchmarkFibPowerMatrix64-8                      3000000               516 ns/op
BenchmarkFibPowerMatrix128-8                     1000000              1034 ns/op
BenchmarkFibPowerMatrix1024-8                     200000              8349 ns/op
BenchmarkFibPowerMatrixRecursive1-8             300000000                4.45 ns/op
BenchmarkFibPowerMatrixRecursive2-8             300000000                4.79 ns/op
BenchmarkFibPowerMatrixRecursive4-8             50000000                24.0 ns/op
BenchmarkFibPowerMatrixRecursive8-8             30000000                45.0 ns/op
BenchmarkFibPowerMatrixRecursive16-8            20000000                65.9 ns/op
BenchmarkFibPowerMatrixRecursive32-8            20000000                86.2 ns/op
BenchmarkFibPowerMatrixRecursive64-8            20000000               107 ns/op
BenchmarkFibPowerMatrixRecursive128-8           10000000               127 ns/op
BenchmarkFibPowerMatrixRecursive1024-8          10000000               187 ns/op    <--- HERE
```

Our iterative implementation is clearly the fastest at a values of n <= 128, but it doesn't after that, `FibPowerMatrixRecursive`, our logn implementation has a lower _ns/op_. This proves our assertion that while the overhead involved in calculating matrices and recursive function calls slows down calls, as n -> infinity, our logn implementation has faster performance.  Thus at lower values of n, it is more beneficial to use `FibIterative`.  But once n grows large enough, the recursive matrix implementation should be used.
