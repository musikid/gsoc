# Rewrite PJDFSTest suite (Draft)

## What is the project?

This project aims to rewrite the PJDFSTest suite.
Today, the tests are written in a mix of shell script and C.
This approach has provided some flexibility and usability, allowing to use syscalls within a shell environment.
However, it also has disadvantages, the main ones being performance, code duplication and
higher entry barrier for potential contributors.
We want to improve the test suite, mainly by switching to a unique language.
After further discussions, we agreed on using Rust, given its numerous advantages,
particularly fearless multithreading, low-level handling and safety.
I aim to write first a single-threaded test runner, along with fixtures to be shared between the tests.
Then, as a secondary objective, I will also add ATF support to rely on Kyua test runner,
to inherit from its high quality reporting.
Finally, I will try to add support for multithreading to our test runner.

#### TL;DR Rewrite the tests in Rust, with a custom-built test runner for running them standalone, and rely on Kyua for running tests along with ATF support to get reports.

## Architecture

### Test collection

The project will not rely on the Rust testing framework,
therefore we will to need to do the test collection ourselves.
Since we plan to adopt an approach similar to Criterion, we will need to write procedural macros.
Criterion collects the tests in a group (`criterion_group!`), which is turned into a function,
to finally aggregate all these functions into the main one (`criterion_main!`).
We want to adopt a similar approach, with some nuances however.
Since we want to be able to list the tests, collecting them in a function seems more troublesome.
Instead, we will collect them in a slice, along with/within a structure for easing listing them.


### Layout

Currently, tests are organized by syscalls, and the rewrite should adopt a similar approach.
Like explained in the previous [section](#test-collection), we declare the tests by groups.

For example:

**symlink/mod.rs**
```rust
pjdfs_group!(symlink, return_enoent, return_eaccess);
```

**main.rs**
```rust
pjdfs_main!(symlink);
```
