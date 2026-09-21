# Complexity Analysis and Performance Report

## Duplicate detection

The naive version compares each value with the values after it. It returns as
soon as it finds a match. In the worst case there are no duplicates, so it does
about n * (n - 1) / 2 comparisons. Time is O(n^2) and extra space is O(1).

The efficient version stores values in an unordered_set. If insertion says the
value already exists, it returns true. Average time is O(n) and extra space is
O(n). An early duplicate can make either version finish much sooner.

## Most frequent value

The naive version counts a value by scanning the whole array, and repeats this
for every value. Time is O(n^2) and extra space is O(1).

The efficient version uses an unordered_map to count each value. It then checks
the map for the largest count. Average time is O(n) and extra space is O(n),
or more precisely O(d) for d different values. Both versions choose the smaller
value when counts tie. Both throw invalid_argument for an empty array because
there is no value to return.

## Common distinct elements

Let n be the left array size and m be the right array size. The naive version
scans the right array for each left value. It keeps a vector of values already
counted so repeated values only count once. There can be at most m values in
that vector, so the extra scan still fits O(n * m) time for nonempty arrays.
Extra space is O(k), where k is the number of distinct common values. With an
empty right array, the code still visits the n left values, taking O(n).

The efficient version puts the right values into an unordered_set and checks
each left value. A second set prevents counting a common value twice. Average
time is O(n + m) and extra space is O(m), since k is at most m.

Hash operations take constant time on average. With many collisions, duplicate
and frequency processing can take O(n^2), and common-element processing can
take O((n + m)^2) as a loose worst-case bound.

## Local benchmark

These are actual measurements on Windows with an AMD Ryzen 7 9700X and
w64devkit GCC 16.2.0. The Makefile's default flags were used (C++17, warnings,
no optimization flag). Run correctness tests before collecting timings:

```bash
make test
make build/benchmark_app
./build/benchmark_app 10000 5 > docs/benchmark_results.csv
```

Each table entry is the median of five trials, in milliseconds. Raw nanosecond
measurements are in [benchmark_results.csv](benchmark_results.csv).

| Problem | Input size | Naive (ms) | Efficient (ms) |
| --- | ---: | ---: | ---: |
| Duplicate | 1,000 | 1.956 | 0.081 |
| Duplicate | 5,000 | 49.277 | 0.461 |
| Duplicate | 10,000 | 196.716 | 0.903 |
| Frequency | 1,000 | 2.448 | 0.026 |
| Frequency | 5,000 | 60.974 | 0.127 |
| Frequency | 10,000 | 243.815 | 0.263 |
| Common | 1,000 | 0.507 | 0.150 |
| Common | 5,000 | 12.231 | 0.822 |
| Common | 10,000 | 48.788 | 1.600 |

Duplicate inputs have no duplicates, which forces the full pair scan. Frequency
inputs repeat seven values. Common inputs are two equal-sized arrays of unique
values, with half of their values overlapping; input_size is the size of each
array. This avoids always finding matches near the beginning of the right array.

The benchmark checks that the two answers agree before timing each input.
Data creation, correctness checks, and CSV printing are outside the timed block.
It uses steady_clock and keeps each result in a volatile variable so the call's
result is used. The small timing helper adds some overhead to every measurement.

## What the results show

From 1,000 to 10,000 elements, the naive times grew about 100 times. The hash
versions grew about 10 to 11 times. At 10,000 elements the hash versions were
about 218 times faster for duplicates, 927 times faster for frequency, and
30 times faster for common elements. This agrees with quadratic versus linear
growth for these inputs. The extra memory avoids repeated scans.

The experiment stops at 10,000 because the naive algorithms get slow quickly.
A million elements would require roughly 10,000 times the work of 10,000
for a quadratic algorithm. Timings depend on the machine, compiler options,
input data, and background programs, so these numbers are not universal.
