# semi-functional-options
A different pattern for functional configuration in golang

DRAFT.

Note: maybe this should be called "better functional variadic configurational"  Yuck.

Note: skip debate on whether functional configuration is a good idea as compared to alternatives.

Note: https://www.youtube.com/watch?v=5uM6z7RnReE&t=1035s

# The OG Functional Options

You've seen this:

https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

which was based on

https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html

Fantastic.  Except.. 

* How do you share options between packages? 
* How do you take a configuration and turn it back into the functional args that created it?
* How do you check that one configuration is the same as another?
* How do you merge two configurations?
* How do you save a configuration?

* It's a lot of code! Well not really but it's 3 - 4 lines of code that are all identical.  Someone smart should write some type of generator using type annotations (similar to JSON).
* Options are tied to a specific configuration structure.  This makes it hard to share options.  One can probably figure out some clever way aroudn this using embedded structs, but then it's More Code!

Many of these points could be solved with More Code!

These points might not matter at all in a small project.

## dsnet's Crazy Experiment in JSON2

So, Golang's JSON functionality needs a rewrite!  

https://github.com/golang/go/discussions/63397

The proposal uses functional options, which would be a first in the golang Standard Library.  But they work differently.  Very differently.

Some context:

dsnet works at Tailscale which making networking stuff that runs in very contrained devices (think: VPN on your phone). Im surprised golang works at all in these environments!

First some design goals (I'm inferring)

* No Reflection (Using reflect dramatically changes the output of the final binary).
* No Heap
* Maximum performance.
* Next to mix and match parts of the functionality
* Need to be able to provide old broken v1 functionality.

The first three are certainly satified with a struct.  The last two are what pushed the design to functional options.

"""
If v1 did not exist, then I would argue for the use of configuration structs.
"""


Solution:

The version in JSON2 is some work to read since it's split into multiple packages, and many of the options are implimented as flags.

But a simplified (and incomplete) version is:

```go

// here's your private/internal configuration struct
type config struct {
        indent string
}

// every option needs to impliment this signature
// so we can identify them
type Option interface {
  function bogus()
}

// every option has it's own type!
type Indent string

// that implements the Option interface
func (Indent) bogus() {}

// here's the functional option
func WithIndent(s string) Option {
   return Indent(s)
}
```

Note:
* Every option has a unique `type`
* Every unqiue type impliments the `Option` interface using a bogus method.

`WithIndent` returns a `Indent(string)`.  Very clever.

But given a list of Options how do you translate it back to a configuration struct?
 
A simplified version of looking up an option.

```go
// to turn the options back into config struct
// use a type switch and cast.
func collectOptions(c *config, opts []Option) {
    for _, opt := range opts {
        switch val := opt.(type) {

        // For each option...
        // Explicity set the internal configuration
        case Indent:
            c.indent = string(val)
   }
}
```


We replaced the function closure with a custom type.








