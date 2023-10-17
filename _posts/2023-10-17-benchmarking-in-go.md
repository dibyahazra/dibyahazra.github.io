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
ok      benchmarking-example        3.430s
```

So this is how you run a benchmark, it tells you approximately how much time it's taking for each iteration which is about 6 nanoseconds right now and it tells us that we're doing 0 allocations.

#### Profiling
Now in addition to this we can easily pass CPU profile and that's now gonna record a profile that only includes my current function (`rand.Int()` in this case). 

```sh
$ go test -bench . -benchmem -cpuprofile prof.cpu

BenchmarkRandInt-8  216021757  5.111 ns/op  0 B/op  0 allocs/op
PASS
ok      benchmarking-example        4.722s
```

One important note is that when you use `-cpuprofile` or `-memprofile` go also builds the `test` binary and that's because the profile just has binary data with no information about what symbols and so on. The binary is used to find the symbol name for those locations. So you run go tool pprof by passing the binary first and then the profile.
```sh
$ go tool pprof .\benchmarking-example.test.exe .\prof.cpu

Duration: 4.73s, Total samples = 3.31s (70.05%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```
Now we can try out different profiling commands to investigate the performance of our function. Some of the frequestly used commands are `list <function_name>, top10, top5` etc. I will cover Profiling in much more detail later in a separate post (probably).

### Example
So we briefly learnt how to benchmark and profile Go code but lets understand it further with a real world example. We are going to select a coding problem that can be solved in multiple ways using Go. We will benchmark and profile each of the approaches and pick the best one in terms of CPU and Memory utilization.

#### Problem Statement
Lets say there is a Json file with the following nested Json data
```json
{
    "data": [
        {"name": "rohit", "dob": "10/11/1995"},
        {"name": "neha", "dob": "12/10/1993"},
        ...100 more records
    ]
}
```
Now the task is to read this file and trim the json key `"data"` and write to a new file.
So the expected json file would look like
```json
[
    {"name": "rohit", "dob": "10/11/1995"},
    {"name": "neha", "dob": "12/10/1993"},
    ...100 more records
]
```

#### Solution
There are multiple ways we can solve this problem so we are going to iterate through each of the possible solutions and benchmark all these approaches and pick the best approach. At first we will create two go files lets call them `task.go` where we put our solution code and `task_test.go` where we put our benchmarking code and one json file `data.json` where we put some dummy data.

- Deserialization

We can unmarshal the file, take only the `data` key and then marshal it back.

```go
func Deserialize() {
	data, _ := os.ReadFile("data.json")
	var m map[string]interface{}

	json.Unmarshal(data, &m)

	if v, ok := m["data"]; ok {
		res, _ := json.Marshal(v)
		os.WriteFile("result-deserialize.json", res, 0644)
	}
}

func BenchmarkDeserialize(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Deserialize()
	}
}
```
- Strings.Trim

Take the string representation of the data and trim the cutset from the beginning and end of the string.
```go
func Trim() {
	data, _ := os.ReadFile("data.json")
	str := strings.Trim(string(data), "{}\n\r\"dat: ")
	os.WriteFile("result-replace.json", []byte(str), 0644)
}

func BenchmarkTrim(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Trim()
	}
}
```

- Regex

I'm not good with Regex, so I have to use it in multiple stages. If you have a better pattern for this task, let me know.
```go
func Regex() {
	data, _ := os.ReadFile("data.json")
	re := regexp.MustCompile(`^{(.*?)\n.*:`)
	str := re.ReplaceAllString(string(data), "")
	re = regexp.MustCompile(`}\z`)
	str = re.ReplaceAllString(str, "")
	os.WriteFile("result-regex.json", []byte(str), 0644)
}

func BenchmarkRegex(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Regex()
	}
}
```

#### Run Benchmarks

Now lets run the benchmarking and find out the best approach!
```sh
$ go test -bench . -benchmem

BenchmarkDeserialize-8  2246  693079 ns/op  52911 B/op  940 allocs/op
BenchmarkTrim-8         2954  406857 ns/op  17608 B/op  10 allocs/op
BenchmarkRegex-8        3165  414273 ns/op  43567 B/op  57 allocs/op
PASS
ok      benchmarking-example    5.983s
```

### Conclusion

So as we can see, BenchmarkDeserialize has run 2246 times and average time per operation is approax 0.69 seconds and 52911 bytes has been allocated per operation and 940 distinct memory allocations occured per op.
So who is the winner then?
We need to pick the approach which has taken lesser time per operation and lesser allocation per operation. So in this case `BenchmarkTrim` is the winner, as a result the 2nd approach using `Strings.Trim` will be the best fit out of the three as it stands out from rest by significant margin!

So this is how we run benchmarking for our Go code. I hope this was helpful and you have enjoyed reading it. If you have any questions or suggestions, do comment down below or message me. Thank you!

### References

- [https://www.youtube.com/watch?v=N3PWzBeLX2M](https://www.youtube.com/watch?v=N3PWzBeLX2M)
- [https://pkg.go.dev/testing#hdr-Benchmarks](https://pkg.go.dev/testing#hdr-Benchmarks)
