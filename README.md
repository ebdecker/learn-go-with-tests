# Learn Go With Tests
This repo is the code and notes I captured while going thru the excellent book [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests). I stated this book as a way to refresh on Go and get back into a true TDD flow. This book accomplishes both. A lot of my notes below are taken like I was capturing notes during a lecture so some are direct copy and paste from the book. However, I summarize each chapter in my own words for future reflection and to make sure I understood the key points and concepts of each subject.

tags: [[golang]] #programming  #softwareengineering #tech 

Link: [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests)

## Chapters

### Hello World
Straight forward
``` erlang
package main

import "fmt"

func main() {
	fmt.Println("Hello, world")
}
```

To make this testable though
```erlang
package main

import "fmt"

func Hello() string {
	return "Hello, world"
}

func main() {
	fmt.Println(Hello())
}
```

and then a new file to test `hello_test.go`

```erlang
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello()
	want := "Hello, world"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

to run unit tests `go test`

`t.Errorf`

We are calling the `Errorf` _method_ on our `t`, which will print out a message and fail the test. The `f` stands for format, which allows us to build a string with values inserted into the placeholder values `%q`. When you make the test fail, it should be clear how it works.

You can read more about the placeholder strings in the [fmt go doc](https://golang.org/pkg/fmt/#hdr-Printing). For tests, `%q` is very useful as it wraps your values in double quotes.

Ended up refactoring the code and tests into the follow:

```erlang
func TestHello(t *testing.T) {
	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Eythan")
		want := "Hello, Eythan"
		assertCorrectMessage(t, got, want)
	})

	t.Run("empty string defaults to 'world'", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"
		assertCorrectMessage(t, got, want)
	})

}

func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```


We've refactored our assertion into a new function. This reduces duplication and improves the readability of our tests. We need to pass in `t *testing.T` so that we can tell the test code to fail when we need to.

For helper functions, it's a good idea to accept a `testing.TB` which is an interface that `*testing.T` and `*testing.B` both satisfy, so you can call helper functions from a test, or a benchmark.

`t.Helper()` is needed to tell the test suite that this method is a helper. **By doing this, when it fails, the line number reported will be in our _function call_ rather than inside our test helper.** This will help other developers track down problems more easily. 

When you have more than one argument of the same type (in our case two strings) rather than having `(got string, want string)` you can shorten it to `(got, want string)`.


#### Summary / Thoughts
Really great hello world example. Started simple and slowly built in more and more concepts and showed the importance of testing. Tests in [[golang]] make sense now to read. Showed good examples of refactoring and keeping code easy to read and splitting out large functions. Really like the start of this book

---
### Integers
url: https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/integers
- 1 package per directory. 
- You can create testable examples of your functions for documentation. https://blog.golang.org/examples

Example test for the integer func we created in this exercise 
``` erlang
func ExampleAdd() {

    sum := Add(1, 5)

    fmt.Println(sum)

    // Output: 6

}
```

NOTE: The output comment is needed. Without, the test will compile but it wont actually execute with `go test`

short chapter, but what was covered:
- More practice of the TDD workflow
    
- Integers, addition
    
- Writing better documentation so users of our code can understand its usage quickly
    
- Examples of how to use our code, which are checked as part of our tests

---
### Iteration
messing around with for loops. Pretty standard. Already covered this in my [[golang]] notes

`+=` is the Add AND Assignment operator

``` erlang
const repeatCount = 5

func Repeat(character string) string {
	var repeated string
	for i := 0; i < repeatCount; i++ {
		repeated += character
	}
	return repeated
}
```

Benchmarking: https://pkg.go.dev/testing#hdr-Benchmarks
``` erlang
func BenchmarkRepeat(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Repeat("a")
	}
}
```

The `testing.B` gives you access to the cryptically named `b.N`.

When the benchmark code is executed, it runs `b.N` times and measures how long it takes.

#### Summary / Thoughts

2 short chapters today. Reviewed for loops, continued the TDD discipline. Learned about benchmarking and example functions in tests. 

---
### Arrays and slices

Let's introduce [`range`](https://gobyexample.com/range) to help clean up our code

```
func Sum(numbers [5]int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

`range` lets you iterate over an array. On each iteration, `range` returns two values - the index and the value. We are choosing to ignore the index value by using `_` [blank identifier](https://golang.org/doc/effective_go.html#blank).

An interesting property of arrays is that the size is encoded in its type. If you try to pass an `[4]int` into a function that expects `[5]int`, it won't compile. They are different types so it's just the same as trying to pass a `string` into a function that wants an `int`.

Go's built-in testing toolkit features a [coverage tool](https://blog.golang.org/cover)
Use `go test -cover` to use built in coverage tool

use [`reflect.DeepEqual`](https://golang.org/pkg/reflect/#DeepEqual) which is useful for seeing if _any_ two variables are the same.

``` erlang
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

It's important to note that `reflect.DeepEqual` is not "type safe" - the code will compile even if you did something a bit silly


We have covered

- Arrays
    
- Slices
    
    - The various ways to make them
        
    - How they have a _fixed_ capacity but you can create new slices from old ones using `append`
        
    - How to slice, slices!
        
    
- `len` to get the length of an array or slice
    
- Test coverage tool
    
- `reflect.DeepEqual` and why it's useful but can reduce the type-safety of your code
    

We've used slices and arrays with integers but they work with any other type too, including arrays/slices themselves. So you can declare a variable of `[][]string` if you need to.

#### summary / thoughts
I already had good notes on slices and arrays but this was a good review and continued the TDD design and ways to interact with arrays and slices.

---
### Structs
[A struct](https://golang.org/ref/spec#Struct_types) is just a named collection of fields where you can store data

 Basic strut 
 ``` erlang
 type Rectangle struct {

    Width  float64

    Height float64

}
```

basically classes in object oriented languages

A **method** is a function with a receiver. A method declaration binds an identifier, the method name, to a method, and associates the method with the receiver's base type.

Methods are very similar to functions but they are called by invoking them on an instance of a particular type. Where you can just call functions wherever you like, such as `Area(rectangle)` you can only call methods on "things".

When your method is called on a variable of that type, you get your reference to its data via the `receiverName` variable. In many other programming languages this is done implicitly and you access the receiver via `this`.

It is a convention in Go to have the receiver variable be the first letter of the type.

``` erlang
r Rectangle
```

[Interfaces](https://golang.org/ref/spec#Interface_types) are a very powerful concept in statically typed languages like Go because they allow you to make functions that can be used with different types and create highly-decoupled code whilst still maintaining type-safety.

In Go **interface resolution is implicit**. If the type you pass in matches what the interface is asking for, it will compile.

[Table driven tests](https://go.dev/wiki/TableDrivenTests) are useful when you want to build a list of test cases that can be tested in the same manner.

In [Test-Driven Development by Example](https://g.co/kgs/yCzDLF) Kent Beck refactors some tests to a point and asserts:

> The test speaks to us more clearly, as if it were an assertion of truth, **not a sequence of operations**

Things covered in this chapter:

- Declaring structs to create your own data types which lets you bundle related data together and make the intent of your code clearer
    
- Declaring interfaces so you can define functions that can be used by different types ([parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism))
    
- Adding methods so you can add functionality to your data types and so you can implement interfaces
    
- Table driven tests to make your assertions clearer and your test suites easier to extend & maintain
#### Summary / Thoughts

structs are straight forward, interfaces make sense but will have to remember its not declared like it is in other languages. Methods and the way receivers are set will take a lot of getting used to, again straight forward but its done differently than languages I am used to. Really like the table driven tests in this example, very readable and easy to iterate on.

---
### Pointers and Errors

In Go if a symbol (variables, types, functions et al) starts with a lowercase symbol then it is private _outside the package it's defined in_.

[Pointers](https://gobyexample.com/pointers) let us _point_ to some values and then let us change them. So rather than taking a copy of the whole Wallet, we instead take a pointer to that wallet so that we can change the original values within it.

``` erlang
func (w *Wallet) Deposit(amount int) {
	w.balance += amount
}

func (w *Wallet) Balance() int {
	return w.balance
}
```
These pointers to structs even have their own name: _struct pointers_ and they are [automatically dereferenced](https://golang.org/ref/spec#Method_values).

`nil` is synonymous with `null` from other programming languages. Errors can be `nil` because the return type of `Withdraw` will be `error`, which is an interface. If you see a function that takes arguments or returns values that are interfaces, they can be nillable.

Like `null` if you try to access a value that is `nil` it will throw a **runtime panic**. This is bad! You should make sure that you check for nils.

to install errcheck cli its `go install github.com/kisielk/errcheck@latest`

Pointers

- Go copies values when you pass them to functions/methods, so if you're writing a function that needs to mutate state you'll need it to take a pointer to the thing you want to change.
    
- The fact that Go takes a copy of values is useful a lot of the time but sometimes you won't want your system to make a copy of something, in which case you need to pass a reference. Examples include referencing very large data structures or things where only one instance is necessary (like database connection pools).
nil

- Pointers can be nil
    
- When a function returns a pointer to something, you need to make sure you check if it's nil or you might raise a runtime exception - the compiler won't help you here.
    
- Useful for when you want to describe a value that could be missing

Errors

- Errors are the way to signify failure when calling a function/method.
    
- By listening to our tests we concluded that checking for a string in an error would result in a flaky test. So we refactored our implementation to use a meaningful value instead and this resulted in easier to test code and concluded this would be easier for users of our API too.
    
- This is not the end of the story with error handling, you can do more sophisticated things but this is just an intro. Later sections will cover more strategies.
    
- [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)

#### Summary / Thoughts
Go in receiver functions only copies values, you have to point to the object specifically to change the actual object and its values. I already knew this but this chapter and its examples do a good job of explaining how this actually works and why its important in practice.  starting to get used to the function syntax of 
`func (r receiver type) functionName(args FunctionArgs) returnType`

This chapter and previous go code i reviewed confirms for me that almost anything that returns something should return an error type, and then the caller downstream can determine how to handle error cases if one is returned by your function. Pretty basic error handling in this section but the basics are clear for now. Curious when this book will get into panics and more advanced error handling

---

### Maps

Declaring a Map is somewhat similar to an array. Except, it starts with the `map` keyword and requires two types. The first is the key type, which is written inside the `[]`. The second is the value type, which goes right after the `[]`
`map[string]string{"test": "this is just a test"}`

```erlang
func (d Dictionary) Search(word string) (string, error) {
	definition, ok := d[word]
	if !ok {
		return "", errors.New("could not find the word you were looking for")
	}

	return definition, nil
}
```

In order to make this pass, we are using an interesting property of the map lookup. It can return 2 values. The second value is a boolean which indicates if the key was found successfully.

This property allows us to differentiate between a word that doesn't exist and a word that just doesn't have a definition.

An interesting property of maps is that you can modify them without passing as an address to it (e.g `&myMap`)

This may make them _feel_ like a "reference type", [but as Dave Cheney describes](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it) they are not.

> A map value is a pointer to a runtime.hmap structure.

So when you pass a map to a function/method, you are indeed copying it, but just the pointer part, not the underlying data structure that contains the data.

you should never initialize a nil map variable:


``` erlang
var m map[string]string
```

Instead, you can initialize an empty map or use the `make` keyword to create a map for you:


```erlang 
var dictionary = map[string]string{}

// OR

var dictionary = make(map[string]string)
```

Both approaches create an empty `hash map` and point `dictionary` at it. Which ensures that you will never get a runtime panic.

Go has a built-in function `delete` that works on maps. It takes two arguments. The first is the map and the second is the key to be removed.

```erlang
func (d Dictionary) Delete(word string) {
	delete(d, word)
}
```

Learned about:
- Create maps
    
- Search for items in maps
    
- Add new items to maps
    
- Update items in maps
    
- Delete items from a map
    
- Learned more about errors
    
    - How to create errors that are constants
        
    - Writing error wrappers
#### Summary / Thoughts

Maps are pretty straight forward, just need to remember Go has built in functionality for things like `delete` and maps do not require a pointer in receiver functions to update the actual map and not just a copy. Also like the clean way to implement the Go Error interface with the error constants used in the code in this chapter.

---
### Dependency Injection

Went over a basic example of dependency injection in Go. Going back to the hello world classic example, how we can use an interface to test what is exactly written to the console. The foundation of this example is the `io.Writer` interface that several things in the standard library implement including `os.Stdout` , `bytest.Buffer1` , and `http.ResponseWriter`

Example of this from this chapter
di.go
```erlang
package main

  

import (

    "fmt"

    "io"

    "log"

    "net/http"

)

  

func Greet(writer io.Writer, name string) {

    fmt.Fprintf(writer, "Hello, %s", name)

}

  

func MyGreeterHandler(w http.ResponseWriter, r *http.Request) {

    Greet(w, "world")

}

  

func main() {

    log.Fatal(http.ListenAndServe(":5001", http.HandlerFunc(MyGreeterHandler)))

}
```

and the tests dependency_test.go
```erlang 
package main

  

import (

    "bytes"

    "testing"

)

  

func TestGreet(t *testing.T) {

    buffer := bytes.Buffer{}

    Greet(&buffer, "Chris")

  

    got := buffer.String()

    want := "Hello, Chris"

  

    if got != want {

        t.Errorf("got %q want %q", got, want)

    }

}
```

short chapter

---
### Mocking
Creating a basic program that counts down from 3 printing each number on a new line and then pausing 1 second, eventually reaching 0 where it will print "Go!"

The backtick syntax is another way of creating a `string` but lets you include things like newlines

_Spies_ are a kind of _mock_ which can record how a dependency is used. They can record the arguments sent in, how many times it has been called, etc. In our case, we're keeping track of how many times `Sleep()` is called

``` erlang
type SpySleeper struct {
	Calls int
}

func (s *SpySleeper) Sleep() {
	s.Calls++
}
```

If your mocking code is becoming complicated or you are having to mock out lots of things to test something, you should _listen_ to that bad feeling and think about your code. Usually it is a sign of

- The thing you are testing is having to do too many things (because it has too many dependencies to mock)
    
    - Break the module apart so it does less
        
    
- Its dependencies are too fine-grained
    
    - Think about how you can consolidate some of these dependencies into one meaningful module
        
    
- Your test is too concerned with implementation details
    
    - Favour testing expected behaviour rather than the implementation
        
    

Normally a lot of mocking points to _bad abstraction_ in your code.

#### Summary / Thoughts
Chapter covered in depth how mocking can work. Go is pretty powerful in how it handles implicit interfaces, allowing you to mock out almost anything if needed. I was familiar with dependency injection already so a lot of this was review but a good review none the less. 

---

### Concurrency

To tell Go to start a new goroutine we turn a function call into a `go` statement by putting the keyword `go` in front of it: `go doSomething()`.

Go routines are typically used with anonymous functions. You can set a parameter and an argument on an anonymous function like so:

```erlang
package concurrency

import (
	"time"
)

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func(u string) {
			results[u] = wc(u)
		}(url)
	}

	time.Sleep(2 * time.Second)

	return results
}
```
By giving each anonymous function a parameter for the url - `u` - and then calling the anonymous function with the `url` as the argument, we make sure that the value of `u` is fixed as the value of `url` for the iteration of the loop that we're launching the goroutine in. `u` is a copy of the value of `url`, and so can't be changed.

The above code introduces a race condition however. We can solve this data race by coordinating goroutines using _channels_. Channels are a Go data structure that can both receive and send values. These operations, along with their details, allow communication between different processes.

In this case we want to think about the communication between the parent process and each of the goroutines that it makes to do the work of running the `WebsiteChecker` function with the url.

``` erlang
package concurrency

type WebsiteChecker func(string) bool
type result struct {
	string
	bool
}

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)
	resultChannel := make(chan result)

	for _, url := range urls {
		go func(u string) {
			resultChannel <- result{u, wc(u)}
		}(url)
	}

	for i := 0; i < len(urls); i++ {
		r := <-resultChannel
		results[r.string] = r.bool
	}

	return results
}
```

Alongside the `results` map we now have a `resultChannel`, which we `make` in the same way. `chan result` is the type of the channel - a channel of `result`. The new type, `result` has been made to associate the return value of the `WebsiteChecker` with the url being checked - it's a struct of `string` and `bool`. As we don't need either value to be named, each of them is anonymous within the struct; this can be useful in when it's hard to know what to name a value.

Now when we iterate over the urls, instead of writing to the `map` directly we're sending a `result` struct for each call to `wc` to the `resultChannel` with a _send statement_. This uses the `<-` operator, taking a channel on the left and a value on the right:

```
// Send statement
resultChannel <- result{u, wc(u)}
```

The next `for` loop iterates once for each of the urls. Inside we're using a _receive expression_, which assigns a value received from a channel to a variable. This also uses the `<-` operator, but with the two operands now reversed: the channel is now on the right and the variable that we're assigning to is on the left:

```
// Receive expression
r := <-resultChannel
```

We then use the `result` received to update the map.

By sending the results into a channel, we can control the timing of each write into the results map, ensuring that it happens one at a time. Although each of the calls of `wc`, and each send to the result channel, is happening concurrently inside its own process, each of the results is being dealt with one at a time as we take values out of the result channel with the receive expression.

the above code with the send and receiver statements are basically appends  handling concurrency and can also be used to feed another variable or process. Almost seems like a queue, FIFO.

This section covered:
- _goroutines_, the basic unit of concurrency in Go, which let us manage more than one website check request.
    
- _anonymous functions_, which we used to start each of the concurrent processes that check websites.
    
- _channels_, to help organize and control the communication between the different processes, allowing us to avoid a _race condition_ bug.
    
- _the race detector_ which helped us debug problems with concurrent code

#### Summary / thoughts

I already was somewhat familiar with go routines, its one of the primary strengths of the language, but these examples did a good job of explaining why you would use an anonymous function and how race conditions can happen, and if not accounted for, provide false positives in tests.  Was introduced to channels which can be an help control communication between 2 processes, a primary tool for avoiding race conditions. Was not able to get the race detector working on my windows desktop but i will try it on macbook soon.

---

### Select

You can use `httptest` to mock up external http servers for unit testing. Example:
``` erlang
func TestRacer(t *testing.T) {

	slowServer := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(20 * time.Millisecond)
		w.WriteHeader(http.StatusOK)
	}))

	fastServer := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
	}))

	slowURL := slowServer.URL
	fastURL := fastServer.URL

	want := fastURL
	got := Racer(slowURL, fastURL)

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}

	slowServer.Close()
	fastServer.Close()
}
```

**Super important defer**
**By prefixing a function call with `defer` it will now call that function _at the end of the containing function_.

Sometimes you will need to clean up resources, such as closing a file or in our case closing a server so that it does not continue to listen to a port.

You want this to execute at the end of the function, but keep the instruction near where you created the server for the benefit of future readers of the code.**
example:
```erlang
func TestRacer(t *testing.T) {

	slowServer := makeDelayedServer(20 * time.Millisecond)
	fastServer := makeDelayedServer(0 * time.Millisecond)

	defer slowServer.Close()
	defer fastServer.Close()

	slowURL := slowServer.URL
	fastURL := fastServer.URL

	want := fastURL
	got := Racer(slowURL, fastURL)

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}
}

func makeDelayedServer(delay time.Duration) *httptest.Server {
	return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(delay)
		w.WriteHeader(http.StatusOK)
	}))
}
```

#### Select 

you can wait for values to be sent to a channel with `myVar := <-ch`. This is a _blocking_ call, as you're waiting for a value.

`select` allows you to wait on _multiple_ channels. The first one to send a value "wins" and the code underneath the `case` is executed.

```erlang
func Racer(a, b string) (winner string) {
	select {
	case <-ping(a):
		return a
	case <-ping(b):
		return b
	}
}

func ping(url string) chan struct{} {
	ch := make(chan struct{})
	go func() {
		http.Get(url)
		close(ch)
	}()
	return ch
}
```

#### Summary / thoughts
### `select`

- Helps you wait on multiple channels.
    
- Sometimes you'll want to include `time.After` in one of your `cases` to prevent your system blocking forever.

Great example in this chapter on how select helps you determine which channel to work with first in a scenario like the website racer. Easy to follow, drove home previous points about making sure you make() all channels so panics are not introduced trying to block a nil channel. Also demonstrates great use of the built in `httptest` library for built in mocking of http responses. 


----
### Reflection
https://go.dev/blog/laws-of-reflection
Reflection in computing is the ability of a program to examine its own structure, particularly through types; it's a form of metaprogramming. It's also a great source of confusion.

As a review, interfaces can take any type. `walk(x interface{}, fn func(string))` will accept any value for `x`.

example:
```erlang
func walk(x interface{}, fn func(input string)) {
	val := reflect.ValueOf(x)

	for i := 0; i < val.NumField(); i++ {
		field := val.Field(i)
		fn(field.String())
	}
}
```

#### Summary / Thoughts

An extremely long chapter over the reflect package and how you can handle all sorts of cases for type handling if your function just accepts `interface{}` as an argument. Also some basic recursive techniques used which was a nice review. I liked the authors final note in the chapter `- Now that you know about reflection, do your best to avoid using it.` and based on how we had to handle every single type in the function gracefully because you never ever know what would get passed in, hes absolutely right. 

---

### Sync
[`sync.WaitGroup`](https://golang.org/pkg/sync/#WaitGroup) which is a convenient way of synchronising concurrent processes.

> A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls Add to set the number of goroutines to wait for. Then each of the goroutines runs and calls Done when finished. At the same time, Wait can be used to block until all goroutines have finished.

```erlang
wantedCount := 1000
	counter := Counter{}

	var wg sync.WaitGroup
	wg.Add(wantedCount)

	for i := 0; i < wantedCount; i++ {
		go func() {
			counter.Inc()
			wg.Done()
		}()
	}
	wg.Wait()
```

The above code above demonstrates an error where multiple go routines are trying to access and update a value at the same time. One way to handle this is with the Sync library's `mutex`

	A Mutex is a mutual exclusion lock. The zero value for a Mutex is an unlocked mutex.

```erlang
type Counter struct {
	mu    sync.Mutex
	value int
}

func (c *Counter) Inc() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.value++
}
```

`go vet` Vet examines Go source code and reports suspicious constructs, such as Printf calls whose arguments do not align with the format string. Vet uses heuristics that do not guarantee all reports are genuine problems, but it can find errors not caught by the compilers. https://pkg.go.dev/cmd/vet

#### Summary / Thoughts
This chapter covered a great use case on why you have to worry about using values in a concurrent environment.  Introduced `mutex` which i was not that familiar with yet but a great standard way to set locks on objects so that another goroutine can not access it until the current one is done. The author also explained why you wouldn't want to expose that functionality in a public api, if other code is coupled to unlocking and locking state without your original intentions, you could encounter weird bugs down the line that would be hard to track down. 

The chapter also introduced `go vet` which is a handy way to check if you have suspicious code that could introduce errors like the scenario above with public unlocking and locking and by copying the mutex of an object from one place another. 

---