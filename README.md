# unittest.h
`unittest.h` is a simple unit-testing framework for C programmng language.  
This library is designed to work with gcc, assuming some specific attributes and nested functions are available.
`unittest.h` is meant to be a simple but better replacement of traditional `abort` tests.
The main features are listed below:

- header only.
- simply designed APIs.
- can handle segfault.
- colored reports.

The name of APIs and the output styles are a bit inspired with [Rove](github.com/fukamachi/rove), one of the unit-testing framework for Common Lisp.  
Note that this library is quite experimentally, so use at your own risks.
Additionally, `unittest.h` is **NOT** thread-safe.

## APIs

### ok (expression)
Tests the evaluation results of expression, assuming the result true (non-zero).

### ng (expression)
Tests the evaluation results of expression, assuming the result false (zero).

### skip (expression)
Does nothing.

### unittest { ... }
Create a special block for defining test cases on the top level.

### setup, teardown
You can define these two functions for preparation and cleanup of tests.
`void setup (void)` is called before all the tests start, `void teardown (void)` is called after all the tests end.

### NDEBUG
If defined, all the APIs are disabled. 
You can pass `-DNDEBUG` flag to compiler to disable tests.

### NOMAIN
If defined, `main` function is automatically added. 

Additionally, note that `unittest.h` installs a signal handler for SIGSEGV. When segmentation fault occurs while running tests, process will be aborted immediately and test will fail. The signal handler basically does nothing, just tries to exit, but if the signal is reported inside tests, you can know in which test case the signal occures.

## Example
Here is a trivial example.

```:.c
#include "unittest.h"

int a = 10;

unittest
{
  ok (a == 10);
  ng (a == 0);
}

int
main (int argc, char *argv[])
{
  ok (a == 10);
  skip (a == 10);
  ng (a == 10);
  return 0;
}
```

You can get output like:

```:
running..... done:
  ✓ "a == 10" expected to be true.
  ✓ "a == 0" expected to be true.
  ✓ "a == 10" expected to be true.
  - "a == 10" skipped.
  ✗ "a == 10" expected to be false.
summary:
  ✗ 1 of 5 tests failed.
  - 1 tests skipped.
  ✓ 3 tests passed.
```

Each lines are colored according to its status.
The exit code will be overwritten according to the test results.  

Here is one more example:

```:.c
#include "unittest.h"

int
main (int argc, char *argv[])
{
  ok (1);               // <- passed
  char *c;
  ok ((c[0] = 2) == 2); // <- segmentation fault
  ok (1);               // <- not reached
  return 0;
}
```

The output will be:

```
running. aborted:
  ✓ "1" expected to be true.
  ✗ "(c[0] = 2) == 2" segmentation fault. aborted.
summary:
  ✗ 1 of 2 tests failed.
  ✓ 1 tests passed.
```

## utest(1)
`utest(1)` is a tool to execute tests easily from the command line. `utest(1)` can directly execute tests from a script file.

### Installation
Place the script bin/utest into anywhere included in PATH. The scipt contains complete copy of `unittest.h` so that you don't need to install the header file on your system.

### Usage

```
$ utest --help
Usage: utest [options] file
Options:
  -D<macro>            define <macro>.
  -h, --help           show this help and exit.
  -I<path>             specify include path.
  -l<lib>              specify shared library to be linked.
  --no-main            tell compiler that no main function defined in files(s).
  -v, --version        show version info and exit.
```

Currently, only very limited compiler options are supported.

Here is an example of test script:

```:.c
/* note that you don't need to include "unittest.h" */
int foo = 100;
unittest
{
  ok (foo < 1000);
}

/* if you omit main function, specify `--no-main` option */
```

You can execute this script as follows:

```
$ utest --no-main your-test-script.c
```

## License
unittest.h is distributed under [MIT License](LICENSE).

## Author
[TANI Kojiro](https://github.com/koji-kojiro) (kojiro0531@gmail.com) 
