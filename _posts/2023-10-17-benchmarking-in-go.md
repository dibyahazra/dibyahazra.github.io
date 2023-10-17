## Benchmarking In Go, with Example

### Introduction
In this article we will see how to use benchmarking to improve performance of your Go code. By following this guide you will be able to select the best performing approach for your task.

#### Benchmarking
So Go makes it really easy to add benchmarks in your `*_test.go` files. You just need to add the Benchmark prefix for your Test function but instead of taking `*testing.T` you take `*testing.B` in the argument and you run the target code/function `b.N` times.

```go
func BenchmarkRandInt(b *testing.B) {
    for i := 0; i < b.N; i++ {
        rand.Int()
    }
}
```

We can run the benchmark by the `go test` command when its `-bench` flag is provided.
We are also gonna add `-benchmem`. What benchmem does is it prints out the number of allocations that are happening for every single iteration of that loop. You'll often find a correlation between the amount of time taken as well as the number of allocations you're doing. Its very often that the more allocations you do the slower your code is.

```sh
$ go test -bench . -benchmem

BenchmarkRandInt-8  195960022  6.002 ns/op  0 B/op  0 allocs/op
PASS
ok      project        3.430s
```

So this is how you run a benchmark, it tells you approximately how much time it's taking for each iteration which is about 6 nanoseconds right now and it tells us that we're doing 0 allocations.

#### Profiling
Now in addition to this we can easily pass CPU profile and that's now gonna record a profile that only includes my current function (`rand.Int()` in this case).

```sh
$ go test -bench . -benchmem -cpuprofile prof.cpu

BenchmarkRandInt-8  216021757  5.111 ns/op  0 B/op  0 allocs/op
PASS
ok      personal        4.722s
```

So we can look at this profile and investigate further
```sh
$ go tool pprof .\prof.cpu

Duration: 1.94s, Total samples = 1.14s (58.69%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```
Now we can try out different profiling commands to investigate the performance of our function. Some of the frequestly used commands are `list <function_name>, top10, top5` etc.

### Example
So we briefly learnt how to benchmark and profile Go code but lets understand it further with a real world example. We are going to select a coding problem that can be solved in multiple ways using Go. We will benchmark and profile each of the approaches and pick the best one in terms of CPU and Memory utiliation.

#### Problem Statement
Lets say there is a large Json file with the folowing nested Json data
```json
{
    "data": {
        [
            {"name": "a100", "dob": "10/11/1995"},
            {"name": "b200", "dob": "12/10/1993"},
            ...10000 more such records
        ]
    }
}
```
Now the task is to modify this file by trimming the json key `"data"`.
So the expected json file would look like
```json
{
   [
        {"name": "a100", "dob": "10/11/1995"},
        {"name": "b200", "dob": "12/10/1993"},
        ...10000 more such records
    ] 
}
```

to be continued...
