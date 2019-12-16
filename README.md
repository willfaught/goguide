# Go coding conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

The sections and conventions are ordered alphabetically.

## General

These conventions closely follow Go idioms, Go best practicies, and the Go Way. They are the consensus of the Go community on how best to write Go. Most who write Go already do these things. Aside from the obvious benefits, it is important to write Go like the Go community so new engineers can more easily understand it.

### MUST comply with go build ./...
### MUST comply with go test ./...
### MUST comply with go vet ./...
### MUST comply with gofmt -s
### MUST comply with goimports
### MUST comply with golint
### MUST support go doc
### MUST support go generate
### MUST support go get
### MUST support go install
### MUST support go mod
### SHOULD comply with https://golang.org/blog
### SHOULD comply with https://golang.org/doc/effective_go.html
### SHOULD comply with https://golang.org/doc/faq
### SHOULD comply with https://golang.org/wiki/CodeReviewComments
### SHOULD follow the example of the Go project
### SHOULD follow the example of the Go standard libraries

## Style

### SHOULD begin one-line comments and end one-line general comments with a space

#### Examples

Good:

```go
// Started earlier
```

```go
/* Started earlier */
```

Bad:

```go
//Started earlier
```

```go
/*Started earlier*/
```

### SHOULD end comments with multiple sentences with punctuation

#### Examples

Good:

```go
// Started earlier. Skip initialization.
```

Bad:

```go
// Started earlier. Skip initialization
```

### SHOULD use sentence case for whole-line comments

#### Examples

Good:

```go
// Started earlier
if stopped() {
```

Bad:

```go
// started earlier
if stopped() {
```

## Testing

### SHOULD NOT use test suites

Some are tempted to use test suites on top of the standard Go test framework, like `testify/suite`.

Some reasons to use plain Go tests:

- It's simpler
- It's fully capable
- It's widely understood
- It's built-in
- It's supported by the Go team, stable, idiomatic, and high-quality
- [It's not a DSL](https://golang.org/doc/faq#testing_framework)

Some reasons not to use `testify/suite`:

- It encourages shared state among test methods, which is difficult to reason about, extend, and parallelize. Most tests should be independent and able to run in parallel. The motivation for using suites goes away as soon as you isolate every test.
- You have to have a driver test in the Go test framework just to get it to work:

```go
func TestFooTestSuite(t *testing.T) {
	suite.Run(t, new(FooTestSuite))
}
```

- You have to write boilerplate just to do simple testing:

```go
type FooTestSuite struct {
	suite.Suite
}

func (suite *FooTestSuite) TestFoo() {
	...
}

func TestFooTestSuite(t *testing.T) {
	suite.Run(t, new(FooTestSuite))
}
```

Compare that to:

```go
func TestFoo(t *testing.T) {
	...
}
```

- It's a third-party, external dependency that we have to learn and maintain

If you look at `suite.Suite`, it doesn't actually do all that much:

```
func (suite *Suite) Assert() *assert.Assertions
func (suite *Suite) Require() *require.Assertions
func (suite *Suite) Run(name string, subtest func()) bool
func (suite *Suite) SetT(t *testing.T)
func (suite *Suite) T() *testing.T
```

It just packages together shared `assert.Assertions`, `require.Assertions`, and `testing.T` values, and provides a helper method for calling `testing.T.Run`. `suite.Run` provides support for setup/teardown hooks. That's it. Go tests already have a `testing.T`, they can already use `assert.Assertions` and `require.Assertions`, and—as illustrated in the `testing` package doc ([1](https://golang.org/pkg/testing/#hdr-Subtests_and_Sub_benchmarks), [2](https://golang.org/pkg/testing/#hdr-Main))—it's trivial enough to do setup/teardown with normal Go code and subtests.
