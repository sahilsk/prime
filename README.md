# prime

This is a small Go package that serves as an example for how to write, test, and document Go libraries. It is covered in detail in the article: ["The anatomy of a Go project"](http://darian.af/post/the-anatomy-of-a-golang-project/)

## Install
    go get github.com/afshin/prime

## Test
    go test -cover github.com/afshin/prime

## Documentation
[![API documentation](https://godoc.org/github.com/afshin/prime?status.svg)](https://godoc.org/github.com/afshin/prime)

## Test coverage
[![Coverage status](https://coveralls.io/repos/afshin/prime/badge.svg)](https://coveralls.io/r/afshin/prime)


==========


[Source](http://darian.af/post/the-anatomy-of-a-golang-project/ "Permalink to The anatomy of a Go project")

# The anatomy of a Go project

There are a lot of resources about [getting started in Go][1] and places where a beginner can [play around][2] and become comfortable with the language. This tutorial is about taking the next steps: building robust projects that are fully documented, tested, and usable by the Go community.

So, let's build something simple and potentially useful: a package with a single public function (`IsPrime`) to check whether an integer is a prime number.

## Step 1: Setting up our project

I'll be hosting this library on GitHub, so I have created [a repository][3] and added the following files:

    $GOPATH/src/github.com/afshin/prime/
        LICENSE            # license file
        prime.go           # (package prime) package code
        prime_test.go      # (package prime) test code
        example_test.go    # (package prime_test) example code
        README.md          # documentation

I've chosen the MIT License, but obviously you can pick any license you prefer.

Go is not a language with overbearing tooling or massive IDEs. It's completely feasible to simply use a text editor and command line tools. I personally love [Sublime Text][4] and [GoSublime][5] because it gives me the best features of an IDE without the bloat.

Unlike most languages, Go is very opinionated about code formatting and documentation. I find that I don't even care whether Go's authors picked a convention that I would have personally picked because it's such a relief that all the Go code in the community follows the same conventions. One of the built-in tools of the Go toolchain is `go fmt`, which you can run on your project to automatically format your code, but if you use GoSublime, your editor will automatically do that.

## Step 2: Coding &amp; documenting

So, without further ado, let's write some code. If your habit is to write comments immediately preceding package and function declarations, then you are already writing Go documentation without realizing it. Go's built-in documentation tool `godoc` generates documentation based on the comments in your source files:

### prime.go __

    // Package prime contains a helper function to check whether
    // an integer is prime. In the future, it may contain other
    // functions, but the API will remain backward-compatible.
    package prime

    // IsPrime checks if candidate is a prime number.
    func IsPrime(candidate uint64) bool {
        // 0 and 1 are not primes.
        if candidate &lt; 2 {
            return false
        }
        // 2 is the only even number that is prime.
        if candidate == 2 {
            return true
        }
        // All other even numbers are not prime.
        if candidate%2 == 0 {
            return false
        }
        var lcv uint64
        // For each divisor checked, close the gap from both sides.
        for lcv = 3; (candidate/lcv)+2 &gt; lcv; lcv = lcv + 2 {
            if candidate%lcv == 0 {
                return false
            }
        }
        return true
    }

This is a straightforward implementation that leaves room for improvement, _e.g._ memoization. But it will suffice.

Running the `godoc` command outputs the documentation we have written so far:

    $ godoc github.com/afshin/prime
    PACKAGE DOCUMENTATION

    package prime
        import "github.com/afshin/prime"

        Package prime contains a helper function to check whether an integer is
        prime. In the future, it may contain other functions, but the API will
        remain backward-compatible.

    FUNCTIONS

    func IsPrime(candidate uint64) bool
        IsPrime checks if candidate is a prime number.

This is a great place to start, but examples would be even nicer. We'll tackle that in next section.

## Step 3: Writing examples &amp; tests

Go has excellent built-in testing support, which can be invoked by the `go test` command. It will check for files that end with `_test.go` and run the tests and examples inside them.

Examples are compiled and run along with tests in order to guarantee that example snippets of code in your documentation never become stale. This is a powerful idea that is baked into Go.

_(As an aside, [Dexy is a sophisticated tool that guarantees your code and documentation stay in sync][6] even when your environment's native capabilities are not as comprehensive as Go's.)_

Here is what our example file contains:

### example_test.go __

    package prime_test

    import (
        "fmt"
        "github.com/afshin/prime"
    )

    func ExampleIsPrime() {
        var candidate uint64 = 5
        if prime.IsPrime(candidate) {
            fmt.Println("Five is prime.")
        } else {
            fmt.Println("Five is not prime.")
        }
        // Output: Five is prime.
    }

There are three things to note in this file:

1. The package name (`prime_test`) is different from our main package (`prime`). Because of this, the example code will not have access to private functions and variables in our library and must `import` it like any other program would. This is a good thing. That means this example file reflects the usage a consumer of our package.

&gt; **Update:** [After a discussion on reddit][7] about whether tests ought to be in their own package or in the package being tested, it seems clear that our tests should be in `package prime` in order to facilitate testing private functions. However, examples are still in a separate package (`package prime_test`) in order to reflect real-world use.
&gt;
&gt; So instead of calling `IsPrime(candidate)`, a real-world consumer of this package would call `prime.IsPrime(candidate)` and that is what the example ought to show.

2. Example function names are important. They must begin with `Example` and be immediately followed by either the function being illustrated (_e.g._ `IsPrime`) or a type and then its member function. Here is [a blog post by Andrew Gerrand that fully explains how to use examples][8]. In particular, you may want to check out the section about [example function names][9].

3. The output comment on the last line of the function is used by the `go test` tool to determine whether the example worked as intended or not. It must be formatted as above.

Now that we've [written our documentation and added an example][10], we can proudly display our shiny documentation badge in our [`README.md`][11]: ![API documentation][12]

[GoDoc][13] is an excellent community resource that automatically publishes documentation for public Go projects; our badge comes directly from them and links to our documentation page.

Tests are functions whose names begin with `Test`, (_e.g._ `TestFirstHundredPrimes`) and accept a single parameter, a pointer to a [`testing.T`][14] instance from the [`testing` package][15] in the standard library. Let's write some tests:

### prime_test.go __

    package prime

    import "testing"

    var firstHundredNonPrimes = [100]uint64{
        0, 1, 4, 6, 8, 9, 10, 12, 14, 15,
        16, 18, 20, 21, 22, 24, 25, 26, 27, 28,
        30, 32, 33, 34, 35, 36, 38, 39, 40, 42,
        44, 45, 46, 48, 49, 50, 51, 52, 54, 55,
        56, 57, 58, 60, 62, 63, 64, 65, 66, 68,
        69, 70, 72, 74, 75, 76, 77, 78, 80, 81,
        82, 84, 85, 86, 87, 88, 90, 91, 92, 93,
        94, 95, 96, 98, 99, 100, 102, 104, 105, 106}

    var firstHundredPrimes = [100]uint64{
        2, 3, 5, 7, 11, 13, 17, 19, 23, 29,
        31, 37, 41, 43, 47, 53, 59, 61, 67, 71,
        73, 79, 83, 89, 97, 101, 103, 107, 109, 113,
        127, 131, 137, 139, 149, 151, 157, 163, 167, 173,
        179, 181, 191, 193, 197, 199, 211, 223, 227, 229,
        233, 239, 241, 251, 257, 263, 269, 271, 277, 281,
        283, 293, 307, 311, 313, 317, 331, 337, 347, 349,
        353, 359, 367, 373, 379, 383, 389, 397, 401, 409,
        419, 421, 431, 433, 439, 443, 449, 457, 461, 463,
        467, 479, 487, 491, 499, 503, 509, 521, 523, 541}

    func TestFirstHundredNonPrimes(t *testing.T) {
        for _, candidate := range firstHundredNonPrimes {
            if IsPrime(candidate) {
                t.Errorf("%d should not be prime.", candidate)
            }
        }
    }
    func TestFirstHundredPrimes(t *testing.T) {
        for _, candidate := range firstHundredPrimes {
            if !IsPrime(candidate) {
                t.Errorf("%d should be prime.", candidate)
            }
        }
    }

The `go test` tool can output a summary of how much of your code is covered by your tests and can even output HTML files that show exactly which lines are covered. Here's a [great blog post by Rob Pike about how to use the code coverage tools in Go][16] if you want to delve into the details.

Let's see how we've done in terms of coverage:

    $ go test -cover github.com/afshin/prime
    ok      github.com/afshin/prime 0.025s  coverage: 100.0% of statements

100%! Sweet!

## Step 4: Setting up continuous integration &amp; coverage

To wrap up, we'll set up coverage integration using [Coveralls][17] to get a code coverage badge in our [`README.md`][11] file.

1. Coveralls is free for open-source projects and you can [sign in with your GitHub account][18]. Sign in and add the repository you want to cover. If you are already using [Travis CI][19] or some other continuous integration tool, you probably already know what to do next.

2. After adding our repository to Coveralls, we need to copy the `repo_token` it generates so that we can tell our continuous integration server how to update Coveralls after each build. We'll be using [Drone][20] because it offers free continuous integration for open-source projects and because it is super simple to set up.

3. Once we have the token, we'll need to add our repository to Drone as well. Again, you can [log in using your GitHub account][21] and add your repository. In the build settings page, you'll need to add the commands and environment variables that the tests need:

        COVERALLS_TOKEN=YOUR-TOKEN-GOES-HERE

4. To generate our coverage report, we'll be using the [`gocov`][22] tool written by [Andrew Wilkins][23]. And in order to communicate from Drone to Coveralls, we'll use the [`goveralls`][24] tool written by [mattn][25]. These are the commands that we'll have Drone execute for each build:

        go get
    go build
    go get github.com/axw/gocov/gocov
    go get github.com/mattn/goveralls
    goveralls -service drone.io -repotoken $COVERALLS_TOKEN

We're all set up! Let's hit the "Build Now" button and see what happens. [Success!][26]

So let's grab our shield code from Coveralls and add that to our [`README.md`][11]: ![Coverage status][27]

## Conclusion

Writing code for personal projects is gratifying in itself, but with some additional effort, your Go packages can reach a much wider audience. Tests, documentation, and coverage help others use your code and inspire confidence in its soundness. I hope this tutorial was helpful. The development toolchain is constantly changing, so it may well be that these recommendations will be obsolete in the not-too-distant future, but the main ideas will likely remain relevant.

[Here is the GitHub repository that contains all the code for this tutorial.][3]

[1]: https://golang.org/doc/install
[2]: https://play.golang.org/
[3]: https://github.com/afshin/prime
[4]: http://www.sublimetext.com/
[5]: https://github.com/DisposaBoy/GoSublime
[6]: http://www.dexy.it/
[7]: https://www.reddit.com/r/golang/comments/3ewsay/the_anatomy_of_a_go_project/ctjgbqg
[8]: https://blog.golang.org/examples
[9]: https://blog.golang.org/examples#TOC_4.
[10]: https://godoc.org/github.com/afshin/prime
[11]: https://github.com/afshin/prime/blob/master/README.md
[12]: https://godoc.org/github.com/afshin/prime?status.svg
[13]: https://godoc.org/-/about
[14]: https://golang.org/pkg/testing/#T
[15]: https://golang.org/pkg/testing/
[16]: https://blog.golang.org/cover
[17]: https://coveralls.io/
[18]: https://coveralls.io/sign-in
[19]: https://travis-ci.org/
[20]: https://drone.io/
[21]: https://drone.io/login
[22]: https://github.com/axw/gocov
[23]: https://github.com/axw
[24]: https://github.com/mattn/goveralls
[25]: https://github.com/mattn
[26]: https://drone.io/github.com/afshin/prime/6
[27]: https://coveralls.io/repos/afshin/prime/badge.svg
  
