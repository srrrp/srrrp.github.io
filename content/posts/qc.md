+++
title = "Integrated Shrinking In QCheck"
date = 2021-07-21
+++


In Property Based Testing (PBT) the programmer specifies desirable properties or invariants for the code under test, and uses a test framework to generate random inputs to find counter-example. For example. *"a reversed list has the same size as the original list"* which can be written as:

```ml
fun l -> List.length (List.reverse l) = List.length l
```

Imagine this test fails for the list `[42; 9079086709; -148; 9; -9876543210]`. Does this counter-example fail the test because there are 5 elements? Or because there are negative numbers? Or maybe due to the big numbers? Many reasons are possible.

To help narrow down the cause of test failures, most PBT libraries provide a feature called *shrinking*. The idea is that once a test fails for a given input, the test engine will try less complex inputs, to find a minimal counter-example. In the example above, if shrinking reduces the minimal failing input to `[-1]` then the developer will more quickly find the root cause: most likely a problem with negative numbers.

This post discusses a type of shrinking called *integrated shrinking* in OCaml — this feature has recently been merged into [QCheck](https://github.com/c-cube/qcheck) and will appear in [QCheck2](https://github.com/c-cube/qcheck/pull/116).


#### Shrinking in QCheck1

In QCheck1, the type of an arbitrary (used to generate and shrink input values) is equivalent to:

```ml
type 'a arbitrary = {
    generate : Random.State.t -> 'a;
    shrink : 'a -> ('a -> unit) -> unit
}
```

- `generate` is used to generate random values
- `shrink` is used in case of test failure to find a smaller counter-example

If the second argument of `shrink` is unsettling, you can simply read it as "the test to run again on smaller values." For example, to aggressively shrink (try all smaller numbers) on an `int`, one could implement `shrink` as such:

```ml
let shrink bigger run_test =
    for smaller = 0 to bigger -- 1 do
    run_test smaller
done
```

For convenience, it is never mandatory in QCheck1 to provide a shrinking function: the `shrink` field is therefore an `option`. By default, no shrinking is done.
