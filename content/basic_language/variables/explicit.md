+++
title = "Explicitly Defined"
+++
If you're coming from a language like Java, you are more than familiar
with explicitly defined and typed variables.  To create a variable you need
to spcify the type and name (and possibly a value).  Go supports explicitly
type variables.

### Declaring a Variable
Although most times you'll prefer implicitly defined and typed variables,
at other times it will be more convenient to implicitly define the variables
type.  For right now, let's look at simple, explicitly defined variables.

```Go
var Name string
var (
	Amount int
	When   time.Time
)

Name = "Joe"
Amount = 35
When = time.Now()
```

The variable can be defined inside a function or at the packae level.
