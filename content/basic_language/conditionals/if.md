+++
title = "if"
+++

The if statement provides Go with conditional execution.  The form of
the if statement is similar to other languages with a true fals test
(a boolean test) and a "then" block and possibly an "else block".  For
example, if a variable <code>today</code> has the current day of the 
week, the following will print a helpful message on Sunday.

```Go
if today == "sunday" {
    fmt.Println("I'm watching the game.")
}
```

The "then" part of the if statement is the block after the if statement.  
The test is any expression that evaluates "true" or "false".  In some 
languages there are truthy values (and by extension falsey values).  For
example, the empty string could also be "false" or any number other than 
zero is considered "true."  In Go, the expression has to be a boolean 
expression.

```Go
// This will NOT compile
score := 0
if score {
    ...
}

// This is the correct version
score := 0
if score > 0 {
    ...
}
```

An if statement can have an "else" clause.  If the expression in the 
first part of the 'if' statement fails, then the else clause is executed.
While the "else" clause is optional, the "then" clause is required.

```Go
score := 0
if score > 0 {
    ...
} else {
    // This is what is executed because the test fails.
    ...
}
```

You can also chain if statements together.  This is often called an "else if"
statement and looks much like you expect.  In the example below, if 
<code>score</code> is 11, the phrase "High score!" prints.  If the 
score is 7, for example, the first test fails (7 is less than 10), but 
the second test succeeds (7 is greater than 5).  If the score is 3, the 
first two tests fail but since 3 is greater than 0, the third test passes
and "Great effort!" is printed.  Finally, if the score is zero the last 
statement is printed.  What happens if the score is -1?  In that case all the test fail and nothing is printed.  [Try it out!](https://play.golang.org/p/-05kTcbwXK)

```Go
if score > 10 {
    // Printed if the score is greater than 10
    fmt.Println("High score!")
} else if score > 5 {
    // Printed if the score is more than 5 and less than or equal to 10
    fmt.Println("Great job!")
} else if score > 0 {
    // Printed if the score is more than 0 but less than or equal to 5
    fmt.Println("Great effort!")
} else {
    // Printed if the score is 0
    fmt.Println("Play again?")
}
```

Let's say there's a function <code>computeScore</code> that calculates
the current score and possibly an error.  We want to check the score 
but we're not worried about the error right now.  Because 
<code>computeScore</code> returns two values, we can't simply write 
<code>if computeScore() > 0</code>.  In this case we can bind the 
values in the first part of the if statement and then test the value in
the second part of the if statement.  The two parts are separated by
a semi-colon.  The variable <code>score</code> only exists in the if
statement, its then clause, an any else clauses (if any).

```Go
// The variable score is only valid for the if statement
if score, _ := computeScore(); score > 0 {
    if score > 10 { 
        fmt.Println("High score!")
    }
    ...
}

// Error! score does not exist out here!
if score == 0 {
    ...
}
```
