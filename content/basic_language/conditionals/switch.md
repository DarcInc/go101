+++
title = "switch"
+++
If the thought of typing lots and lots of "else if" clasues in an
if statement seems tedious, then the switch statement is what you 
crave.  A switch statement is a series of conditional expression
and clauses to execute when a condition expression is true.  It can
take a variable or can use arbitrary expressions.  If you're used 
to the switch statement in C or C++, Go's switch is much more 
flexible.

```Go
dayOfWeek := "monday"
switch dayOfWeek {
case "monday":
    fmt.Println("Someone has a case of the Mondays!")
case "tuesday":
    fmt.Println("Buckle down to work day.")
case "wednesday":
    fmt.Println("Half way there!")
case "thursday":
    fmt.Println("Almost there!")
case "friday":
    fmt.Println("It's the weekend!")
case "saturday": 
    fmt.Println("Let's go play!")
case "sunday":
    fmt.Println("Sob... The weekend's over.")
default:
    fmt.Println("That's not a day of the week.")
}
```

In the example above ([Try It Out!](https://play.golang.org/p/DZeEgYslf2)), 
you get a different message depending on the day of the week.  The value
passed to the switch does not need to be a string.  It could be an integer,
for example.  The variable passed in (<code>dayOfWeek</code>) is compared 
to the value in each of the case statements.  When the values are equal, 
the statements in that case are executed.  Unlike some languages there's 
no 'break' at the end of each case statement.  The default clause (which
is optional) basically allows you to add some statements to execute when
none of the cases match.

Switch statements don't have to refer to a specific variable.  The case 
conditions can be simple boolean expressions.  For example, say we have a 
high score and the user's current score.  The case conditional tests can
compare the scores instead of just testing the score value.  [Try It Out!](https://play.golang.org/p/E0a6vY16sL)

```Go
score := 7
highscore := 10

switch {
case score == 0:
    fmt.Println("Try again!")
case score < highscore:
    fmt.Println("Almost there!")
case score == highscore:
    fmt.Println("You got the high score!")
case score > highscore:
    fmt.Println("You have a new high score!")
}
```

