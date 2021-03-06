# GoAWK: an AWK interpreter written in Go

[![GoDoc](https://godoc.org/github.com/benhoyt/goawk?status.png)](https://godoc.org/github.com/benhoyt/goawk)
[![Build Status](https://travis-ci.org/benhoyt/goawk.svg)](https://travis-ci.org/benhoyt/goawk)

AWK is a fascinating text-processing language, and somehow after reading the delightfully-terse [*The AWK Programming Language*](https://ia802309.us.archive.org/25/items/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) I was inspired to write an interpreter for it in Go. So here it is, pretty much feature-complete and tested against "the one true AWK" test suite.

<!-- [**Read more about how it works and performs here.**](TODO) -->

## Basic usage

To use the command-line version, simply use `go get` to install it, and then run it using `goawk` (assuming `$GOPATH/bin` is in your `PATH`):

    $ go get github.com/benhoyt/goawk
    $ goawk 'BEGIN { print "foo", 42 }'
    foo 42
    $ echo 1 2 3 | goawk '{ print $1 + $3 }'
    4

To use it in your Go programs, you can call `interp.Exec()` directly for simple needs:

    input := bytes.NewReader([]byte("foo bar\n\nbaz buz"))
    err := interp.Exec("$0 { print $1 }", " ", input, nil)
    if err != nil {
        fmt.Println(err)
        return
    }
    // Output:
    // foo
    // baz

Or you can use the `parser` module and then `interp.New()` and `Interp.Exec()` to control execution, set variables, etc:

    src := "{ print $1+$2 }"
    input := "1,2\n3,4\n5,6"

    prog, err := parser.ParseProgram([]byte(src))
    if err != nil {
        fmt.Println(err)
        return
    }
    p := interp.New(nil, nil)
    p.SetVar("FS", ",")
    err = p.Exec(prog, bytes.NewReader([]byte(input)), nil)
    if err != nil {
        fmt.Println(err)
        return
    }
    // Output:
    // 3
    // 7
    // 11

Read the [GoDoc documentation](https://godoc.org/github.com/benhoyt/goawk) for more details.

## Differences from AWK

The intention is for GoAWK to conform to the [POSIX AWK spec](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/awk.html), but this section describes some areas where it's different.

Additional features GoAWK has over AWK:

* It's embeddable in your Go programs. :-)
* I/O is a little bit faster. (TODO: benchmarks)
* The parser supports `'single-quoted strings'` in addition to `"double-quoted strings"`, primarily to make Windows one-liners easier (the Windows `cmd.exe` shell uses `"` as the quote character).

Things AWK has over GoAWK:

* The GoAWK interpreter is significantly slower. (TODO: benchmarks)
* GoAWK has a couple of known syntax quirks, but most real code shouldn't run into them (commented-out tests in interp/interp_test.go). I intend to fix these soon.

## The end

Have fun, and please [contact me](https://benhoyt.com/) if you're using GoAWK or have any feedback!
