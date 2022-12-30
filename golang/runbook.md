# Golang runbook

## How to profile a go program?
[use pprof](https://blog.golang.org/pprof)

## How to tune go GC
To understand how much time go is spening in GC, take a profile using pprof. Understand which objects are occupying most of the heap and tune your code.


