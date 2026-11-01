# Validation and reproduction

Audited upstream base: [`4b21d154b4864531641f90e1b87108575e673d75`](https://github.com/python/cpython/commit/4b21d154b4864531641f90e1b87108575e673d75).

## Minimal reproduction

The following uses `ThreadPool`, which shares the affected iterator
implementation with the process pool:

```python
from multiprocessing.pool import ThreadPool

with ThreadPool(1) as pool:
    for _ in range(10):
        assert list(pool.imap(str, [1], buffersize=1)) == ["1"]
    print(len(pool._taskqueue_buffersize_semaphores))
```

The unmodified audited module leaves 10 entries. The patched module leaves 0.
The private registry is inspected only to demonstrate the retained references.
The same completed-operation behavior was also reproduced with a spawn process
pool, `imap_unordered()`, and chunksizes 1 and 2.

The local host was Windows CPython **3.15.0b2**, loading the audited 3.16
`multiprocessing.pool` source. This was **not** a native build of CPython 3.16.
Running this snippet directly on an unmodified older runtime that lacks
`buffersize` does not test the reported defect.

## Regression coverage

The added native-suite method is
`test_imap_with_buffersize_releases_semaphores` in
`Lib/test/_test_multiprocessing.py`. It consumes ordered and unordered results,
uses chunksizes 1 and 2, covers empty and nonempty inputs, and checks that no
completed registrations remain. Manager pools are skipped because the test
requires access to the pool's internal registry.

Two local executions of the regression method (threads and spawn processes)
produced 16 subtest failures before the patch and none after it. A subsequent
selection of **64 cases** from the source test methods passed, including the
two executions of the new regression test. This selection extracted method
bodies and supplied their parameters in a 3.15-compatible harness; it was not
the native 3.16 regrtest runner.

An additional **68 bounded scenarios** passed: 20 worker/iterable exception
cases, 8 out-of-order result cases, and 40 concurrent completion/shutdown cases.
The local harness used at most two workers simultaneously, finite timeouts,
and temporary directories removed after each run. No test processes remained.

A separate unpatched 2,000-iteration experiment retained 2,000 semaphore objects
and approximately 2.7 MB of additional Python-tracked allocations. This is a
local diagnostic measurement, not a production projection.

The shutdown snapshot change was reviewed against the CPython container
implementation and a controlled iterator interleaving. **No free-threaded
runtime was tested.**

## Suggested native validation

After building this branch with the standard CPython development instructions,
run the regression through the native test runner, for example on Windows:

```powershell
PCbuild\amd64\python.exe -m test test_multiprocessing_spawn -m "*test_imap_with_buffersize_releases_semaphores*"
```

The full multiprocessing suite and a free-threaded build remain useful CI
validation. These native commands were not executed locally.

## Scope

The proposed change concerns retained Python semaphore registrations in
buffered pool iterators. It does not claim that worker processes leak, that
`shutdown()` always hangs, or that unrelated archive behavior is corrected.

AI tools assisted the investigation, implementation, validation harnesses and
explanatory image. The runtime limitations above are part of the reported
evidence.

