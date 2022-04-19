---
mainfont: Fira Sans
---

# GSoC 2022 Proposal | Rewrite PJDFSTest suite

## Introduction

This project aims to rewrite the PJDFSTest suite.
Today, the tests are written in a mix of shell script and C.
This approach has provided some flexibility and usability,
allowing to use syscalls within a shell environment.
However, it also has disadvantages, the main ones being performance,
code duplication and higher entry barrier for potential contributors.
We want to improve the test suite, by switching to a unique language, Rust.
Rust has numerous advantages,
in particular, fearless multithreading, low-level handling and increased safety.

#### TL;DR Rewrite the tests in Rust, with a custom-built test runner for running them standalone, and rely on Kyua for running tests along with ATF support to get reports.

### Mentor: asomers (Alan Somers \<asomers@FreeBSD.org\>)

## whoami?

Name: Sayafdine Said\
GitHub: [musikid](https://github.com/musikid)\
Discord: musikid.#7043\
Matrix: musikid64\
Mail address: [musikid@outlook.com](mailto:musikid@outlook.com)\
Languages: French, English\
Timezone: Paris, France (UTC+2)

Currently a computer science student at Sorbonne University,
I am proficient in Rust, Python, C/C++ and JavaScript. 
I run Arch Linux and FreeBSD, so I am pretty familiar with the Unix environment.
I wrote fixes for some open source projects, but never been more involved.
I also did a bit of yak shaving,
[fancy](https://github.com/musikid/fancy.git) being the most complete project I produced.
This GSoC is the perfect occasion to be involved in the open source community,
particularly for the FreeBSD project, that I love!

## Summary

We want to write a Rust binary, which:

- is self-contained (embed all the tests),
- can execute all the tests (with the test runner),
- can filter according to conditions,
- *is compatible with ATF*.

## Details

### Test collection

The project will not rely on the Rust testing framework,
therefore, we will need to do the test collection ourselves.
As an example, [Criterion](https://docs.rs/criterion/latest/criterion/index.html) uses declarative macros: 

- to collect the tests in a group (`criterion_group!`), which is turned into a function,
- and aggregate all these functions into the main one (`criterion_main!`).

We want to adopt a similar approach, with some nuances, however.

Since we want to be able to list the tests, collecting them in a function seems more troublesome.
Instead, we will collect them in a slice, along with/within a structure for easing listing them.
The structure could be constructed using an attribute macro, or plain syntax if I'm running out of time.

### Test runner

The test runner will not need to have a lot of fancy features. It has to support:

- filtering/skipping tests according to conditions, 
- printing the status,
- reading a configuration file.

### Macros

#### `pjdfs_group!`

Make a group of tests.

#### `pjdfs_main!`

Make the main function.

#### `#[pjdfs_test(conditions)]`

Declare a test.

The attribute might come with arguments to declare conditions.
For example, some tests need to be run as root, 
or some syscalls are not available everywhere.
So, we can imagine an interface analogous to `#[cfg]`.
For example:

```rust
#[pjdfs_test(user = "root")]
fn my_test() {
}
```

In the experimental Python [rewrite](https://github.com/pjd/pjdfstest/pull/48/commits/c08d508282bf2b8c8956e911f1cf2ab93b455072#diff-c6f9a6ca51c3f0551ee9b236b5fba6eec1c3a7ae82720e508642b0404dbc6118R28-R32), 
Python decorators are used.


###### `#[case(args)]`

Declare parameterization.

#### `#[fixture(conditions)]`

Declare a fixture.


We take inspiration from [pytest](https://docs.pytest.org/en/7.1.x/explanation/fixtures.html)
and [rstest](https://docs.rs/rstest/latest/rstest/attr.fixture.html),
and add the fixture to the test's parameters.

The attribute might come with arguments to declare conditions (like `pjdfs_test`). 
Because we want to avoid duplication, when tests share conditions, we can declare a common fixture.

For example:

```rust
#[fixture(user = "root", feature = "posix_fallocate")]
fn posix_fallocate_and_root_user() -> Something {}

#[pjdfs_test]
fn posix_fallocate_1(posix_fallocate_and_root_user: Something) {}

#[pjdfs_test(my_own_condition = "yes")]
fn posix_fallocate_2(posix_fallocate_and_root_user: Something) {}
```

### Configuration file

The format needs further investigation, but we should at least support filtering by syscalls availability.

### *ATF support*

To support ATF, we need to be able to output the tests in ATF metadata format.
The format is fairly simple, consisting only of `key: value` pairs.
For example:

```
ident: test_1
descr: this is a test
require.user: root
```

ATF supports specifying some conditions. However, except for privileges, I did not saw any condition which could intersect with ours.
Thus, the only required keys to support are `ident` and `descr`.
We can also add `require.user`, and eventually timeout if we want to delegate it to `kyua`.

### Layout

Currently, tests are organized by syscalls, and the rewrite should adopt a similar approach.
Like explained in the previous [section](#test-collection), we declare the tests by groups.

For example:

**symlink/mod.rs**
```rust
pjdfs_group!(posix_fallocate, posix_fallocate_1, posix_fallocate_2);
```

**main.rs**
```rust
pjdfs_main!(posix_fallocate);
```

### Command-line arguments

The command-line interface should also be compatible with ATF.
It has a really simple interface, consisting only of two running modes:

- `-l`, to list all the tests and their conditions.
- `[-r resfile] [-s srcdir] [-v var1=value1 [.. -v varN=valueN]] test_case`, to run a test case.

Otherwise, running the program without arguments should call the runner with all the tests meeting the current conditions.
For further configuration, we might also add CLI arguments in addition of the configuration file, however it's not a priority compared to having a configuration file.

## Timeline

After implementing the test collection along with the fixtures, 
I aim to write a single-threaded test runner.
Then, as a secondary objective,
I will add ATF support to rely on Kyua test runner,
to inherit from its high quality reporting.
Finally, I will try to add support for multithreading to our test runner.

Adding multithreading and ATF support are not top priorities, though, 
the main one being to write the tests.

#### NOTE: All the time intervals imply writing the documentation along with the code.

### Community bonding (May 20)

- Get more familiar with the current codebase
- Review the missing syscalls
- Determine the test structure's shape
- Discuss on details

### 1st week (June 13)

- Iterate on the project's design
- Start implementing test collection

### 2nd week - 4th week (June 20)

- Implement fixtures
- Continue implementing test collection
- Implement test macro

### 5th - 7th weeks (July 11)

- Implement test runner
- Continue implementing fixtures
- Start writing the tests

#### Phase 1 evaluation period (July 25)

### 8th - 9th weeks (August 1)

- *Add ATF support*
- Finish implementing fixtures
- Continue writing the tests

### 10th - 12th weeks (August 15)

- *Add multithreading support*
- Continue writing the tests

### 13th week - End (September 4)

- Document extensively
- Write the eventual missing tests
- Unexpected delay


## Relevant links

[https://github.com/pjd/pjdfstest/issues/59](https://github.com/pjd/pjdfstest/issues/59)

[https://github.com/musikid/pytest-atf.git](https://github.com/musikid/pytest-atf.git)

[https://www.freebsd.org/cgi/man.cgi?query=atf-test-program&sektion=1&apropos=0&manpath=FreeBSD+13.0-RELEASE+and+Ports](https://www.freebsd.org/cgi/man.cgi?query=atf-test-program&sektion=1&apropos=0&manpath=FreeBSD+13.0-RELEASE+and+Ports)

[https://www.freebsd.org/cgi/man.cgi?query=atf-test-case&sektion=4&apropos=0&manpath=FreeBSD+13.0-RELEASE+and+Ports](https://www.freebsd.org/cgi/man.cgi?query=atf-test-case&sektion=4&apropos=0&manpath=FreeBSD+13.0-RELEASE+and+Ports)

[https://nexte.st/book/how-it-works.html](https://nexte.st/book/how-it-works.html)

[https://github.com/jmmv/kyua/wiki/About](https://github.com/jmmv/kyua/wiki/About)
