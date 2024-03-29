# RUN

## All go commands except when you need mac binaries 

(i.e. substitute this in for 'go' on the following lines if you don't want to install it locally)
```
docker run --rm -w=/usr/src/app -v $(pwd):/usr/src/app golang:1.12.2 go
```
## go for mac binaries  (
i.e. substitute this in for 'go' on the following lines where you're creating an executable you'll  want to  run on mac)
```
docker run --rm -w=/usr/src/app -v $(pwd):/usr/src/app -e "GOOS=darwin" -e "GOARCH=amd64" golang:1.12.2 go
```
# DOCS

### List
List all packages / modules
```
go list all
```
list all using vendored modules (offline)

```
go list -mod vendor all
```

### Read
package docs

```
go doc math
```
exported function detail docs
```
go doc math.Abs
```
all exported function detail docs
```
go doc -all math
```
Docs on downloaded modules while offline -- need to cd to root of package in vendor dir

```
cd vendor/github.com/spf13/cobra/ && go doc && cd - > /dev/null
```

## MODULES
Make a module
```
go mod init github.com/<account>/<repo>
```
download all the dependencies for a module into a 'vendor' directory
```
go mod vendor
```

## BUILD:
build a package using remote sources --  use the "go for mac binaries" command if you have a mac:
```
go build
```

run 'go build' using the vendor directory --  use the  "go for mac binaries" command if you have a mac
```
go build -mod vendor
```
## TEST:
build and run test files using remote sources:
```
go test
```
using vendored files
```
go test -mod vendor
```

## MORE RESOURCES

repo  of examples
```
git clone git@github.com:mmcgrana/gobyexample.git
```
can run with 'tour' and it will serve locally
```
go get golang.org/x/tour
```

## WORKFLOWS
download the artifacts, build, run the tests

```
go mod vendor
go build -mod vendor
go test
```

build / test cycle
```
go build -mod vendor
go test
```

install an external package -- import the package in a file, then re-run
```
go mod vendor
```

## New repo

Make a module -- if your module produces an executable, the name of
the executable when you run `go build` will be the <repo> name given here
```
go mod init github.com/<account>/<repo>
```

### Write a main and tests

Example `main.go` -- exported function and main function
```
// when the package name is 'main' go will build an executable
package main

// unused imports would be an error
import (
  "fmt"
)

// Note the initial capital--that means this function is exported
func Abs(f64 float64) (ret float64) {
	ret = f64
	if f64 < 0 {
		ret = negate(ret)
	}
  // will return `ret`, because that is named as the return value in the signature
	return
}

func negate(f64 float64) (float64) {
  return f64  * -1
}

// lowercase, no return value
func main() {
  fmt.Println("abs -2.5 =", Abs(-2.5))
}
```

Example `main_test.go`

```
// filename must end in `_test.go`

// same package as code under test
package main

import (
	"testing"
	"fmt"
)

// must start with `Test` and have this signature
func TestAbs(t *testing.T) {
	fmt.Println("testing Abs")
	// Abs is exported because it starts with a capital letter in main.go,
	got := Abs(-1.0)
	if got != 1.0 {
		t.Errorf("Abs(-1.0) = %f; want 1", got)
	}
}

// the part of the name that comes after Test is not important
func TestNegateWorks(t *testing.T) {
	fmt.Println("testing negate")
	// negate is not exported because it starts with a lowercase letter
	// in main.go, but it can still be tested because it's visible
	// within the `main` package this test is in.
	got := negate(-1.0)
	if got != 1.0 {
		t.Errorf("Abs(-1.0) = %f; want 1", got)
	}
	// reassignment uses `=`, not `:=`
	got = negate(1.0)
	if got != -1.0 {
		t.Errorf("Abs(-1.0) = %f; want 1", got)
	}
}
```

Build
```
go build
```

Run -- the executable name is the name of the repo you used in `go mod`
```
./<repo>
```

Test
```
go test
```

## IT'S NOT WORKING

### Arguments and flags
If commands don't seem to be using the flags you set, make sure you've
specified the flags on the correct subcommand:

```
# WRONG: will not use `-mod vendor`; will try to download packages instead of using  local versions:
go list all -mod vendor
# CORRECT: will use `-mod vendor` and list packages from local
go list -mod vendor all
```

### Building:
1. If you want an executable file:
   - The package must be named `main`
   - There must be a `func main() {` (no arguments, no return)
2. These examples assume that all files are in the repo root

### Testing
1. Your test file name must end in `_test.go`
2. Your test file must have the same package name as your src package
3. The functions you're trying to test must be accessible (exported or package-local in the same package
4. These examples assume that all files are in the repo root

### Running
1. During the _build_ step, you must set the GOOS and GOARCH appropriately for where the executable will run.

### Using the `tour`
1. If you're running the tour executable locally, the `run` button may not work, presenting an excellent opportunity to get used to a local dev workflow.

## OPEN QUESTIONS

## How to get complete docs for modules used by an immediate dependency

`cobra` uses `pflag` as `flag`. When I try to get the docs for FlagSet through `cobra`

```
cd vendor/github.com/spf13/cobra/ && go doc flag.FlagSet | less && cd - > /dev/null
```

It doesn't give me all of the functions defined on `FlagSet` in the `pflag` package. Compare

```
cd vendor/github.com/spf13/pflag/ && go doc FlagSet | less && cd - > /dev/null
```

### Helpful bash functions

```
# go list modules. if an argument is provided it will be used as a grep search term
goll () {
  if [[ -z "${1}" ]] ; then
    go list -mod vendor all
  else
    go list -mod vendor all | grep "${1}"
  fi
}

# go docs piped to less, should work offline
gold () {
  # Get out of any project directories that will make
  # us go to the internet for docs. A better solution for this
  # would be appreciated
  pushd ${HOME} 1> /dev/null;
  go doc --all $@ | less -a
  popd 1> /dev/null
}
```
