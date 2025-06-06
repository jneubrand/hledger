hledger test-related files. See also [Developer docs > TESTS].

unittest.hs - main file for a cabal test suite in the hledger package
(run by "cabal test" or "stack test"). Runs the unit tests built in to
all hledger modules. Not used much, we usually run them via hledger's
builtin "test" command instead.

doctest.hs - main file for another cabal test suite. Runs the doctests
embedded in haddock comments in some hledger modules.

The rest of the files here are functional tests, run with [shelltestrunner].
These test the hledger CLI and (indirectly) the hledger-lib package.
They are organised roughly by [component].

shelltestrunner's latest [format 3] is preferred,
though some tests may still be using the older [format 1].
Each command line (beginning with `$`) is a single test,
which usually reads input from a preceding `<` section,
and is followed by expected output,
and any non-standard error output (`>2`) or exit code (`>=`).

Additionally, each test is preceded by a descriptive comment (`#`).
This should begin with an Emacs outshine heading marker (` ** `),
useful for folding and browsing tests in Emacs,
and a test number (`1. `), useful for running individual tests.

A few tests invoke unix commands; these won't run in a Windows CMD shell.

2024-09-30 Note: tests of error output must use regexps for now to work
around ghc 9.10's extra newline in error output: https://gitlab.haskell.org/ghc/ghc/-/issues/25116

[Developer docs > TESTS]: https://hledger.org/TESTS.html
[component]: https://hledger.org/CONTRIBUTING.html#components
[shelltestrunner]: https://github.com/simonmichael/shelltestrunner#readme
[format 1]: https://github.com/simonmichael/shelltestrunner#format-1
[format 3]: https://github.com/simonmichael/shelltestrunner#format-3


Run them all (also builds hledger):

    just functest

See the commands being run:

    $ just -v functest
    ===> Running recipe `functest`...
    $STACK build --ghc-options=-Werror hledger
    time ((stack exec -- shelltest --execdir --exclude=/_ --threads=32  hledger/test/ bin/ -x ledger-compat/ledger-baseline -x ledger-compat/ledger-regress -x ledger-compat/ledger-extra && echo $@ PASSED) || (echo $@ FAILED; false))

Some explanation:

- `stack exec -- ...` ensures you are testing the hledger executable that was just built (it will be first in PATH).
- `--execdir` executes tests within their test file's directory.
- `--exclude=/_` excludes top-level test files whose names begin with underscore.
- `--threads=N` runs tests in parallel which is much faster.
- `-x` is another spelling of --exclude

Run only the tests matching a regular expression (here, the 19th test in the assertions.test file):

    $ just functest -i balance-assertions.*19
    :hledger/test/journal/assertions.test:19: [OK]

             Test Cases  Total      
     Passed  1           1          
     Failed  0           0          
     Total   1           1          

Run a specific test repeatedly as its file is changed:

    $ watchexec -w hledger/test/journal/assertions.test just functest -i balance-assertions.*19
    :hledger/test/cli/query-args.test:1: [OK]

             Test Cases  Total      
     Passed  1           1          
     Failed  0           0          
     Total   1           1          
    :hledger/test/cli/query-args.test:1: [OK]

             Test Cases  Total      
     Passed  1           1          
     Failed  0           0          
     Total   1           1          
      C-c C-c

More shelltestrunner options:

    $ shelltest --help

