# File sizes

While the impact to build times is negligible, this page investigates how generics affect the file sizes of compiled code:

* [**The example**](#the-example): an exemplar case
* [**The benchmark**](#the-benchmark): how does it stack up?
* [**Key takeaways**](#key-takeaways): what it all means


## The example

This page describes how to run `BenchmarkGoBuild`, a Go-based benchmark that inspects the file sizes of:

* a package archive
* a binary executable

Each build varies based on the:

* list type, ex. boxed, generic, typed
* number of distinct type definitions

The packages used by this benchmark are:

* **`./lists/boxed`**: defines `type List []interface{}`
* **`./lists/typed`**: defines zero to many list types based on build tags
* **`./lists/generic`**: defines `type List[T any] []T`

The `typed` and `generic` packages are also subject to the following build tags:

* **`int`**: activates that package's list of `int`
* **`int8`**: activates that package's list of `int8`
* **`int16`**: activates that package's list of `int16`
* **`int32`**: activates that package's list of `int32`
* **`int64`**: activates that package's list of `int64`


## The benchmark

Run the benchmark with the following command:

```bash
docker run -it --rm go-generics-the-hard-way \
  go test -bench GoBuild -run GoBuild -count 1 -v ./06-benchmarks
```

The following table was generated by piping the above command to `hack/b2md.py -t filesize`:

| Artifact type | Number of types | File size (bytes) - typed | File size (bytes) - generic | Increase (bytes) | Increase (%) |
|:-------------:|:---------------:|:-------------------------:|:---------------------------:|:----------------:|:------------:|
| pkg | 0 | 818 | 1194 | 376 | 45.97 |
|  | 1 | 5948 | 11534 | 5586 | 93.91 |
|  | 2 | 10458 | 20214 | 9756 | 93.29 |
|  | 3 | 15004 | 28932 | 13928 | 92.83 |
|  | 4 | 19552 | 37648 | 18096 | 92.55 |
|  | 5 | 24098 | 46278 | 22180 | 92.04 |
| bin | 0 | 1121184 | 1121184 | 0 | 0 |
|  | 1 | 1121184 | 1125280 | 4096 | 0.37 |
|  | 2 | 1125280 | 1125280 | 0 | 0 |
|  | 3 | 1125280 | 1125280 | 0 | 0 |
|  | 4 | 1125280 | 1125280 | 0 | 0 |
|  | 5 | 1125280 | 1125280 | 0 | 0 |

### Validating the artifacts

To validate each of the produced artifacts contain the expected types, use the following commands:

```bash
# Print types in each generic package archive
find -s ./06-benchmarks -name "generic-*-types.a" -type f -print0 | \
xargs -0 -I% sh -c "echo % && go tool objdump -S % | grep -o 'go.shape.int[[:digit:]_]\{1,\}' | sort -ru && echo"
```

```bash
# Print types in each generic binary executable
find -s ./06-benchmarks -name "generic-*-types.bin" -type f -print0 | \
xargs -0 -I% sh -c "echo % && go tool objdump -S % | grep -o 'int[[:digit:]]\{0,\}List' | sort -ru && echo"
```

```bash
# Print types in each typed package archive
find -s ./06-benchmarks -name "typed-*-types.a" -type f -print0 | \
xargs -0 -I% sh -c "echo % && go tool objdump -S % | grep -o 'Int[[:digit:]]\{0,\}List' | sort -ru && echo"
```

```bash
# Print types in each typed binary executable
find -s ./06-benchmarks -name "typed-*-types.bin" -type f -print0 | \
xargs -0 -I% sh -c "echo % && go tool objdump -S % | grep -o 'int[[:digit:]]\{0,\}List' | sort -ru && echo"
```

In all cases the output should indicate a file with:

* 0 types has no match
* 1 type matches `int`
* 2 types match `int` and `int8`
* 3 types match `int`, `int8`, and `int16`
* 4 types match `int`, `int8`, `int16`, and `int32`
* 5 types match `int`, `int8`, `int16`, `int32`, and `int64`

## Key takeaways

* It appears that package archives built from generic code are consistently close to twice as large versus as non-generic counterparts ([golang/go#50438](https://github.com/golang/go/issues/50438))
* At the same time, generic types appear to have no discernable impact on the size of compiled, binary executables

---

Next: [Lessons learned](../07-lessons-learned/)
