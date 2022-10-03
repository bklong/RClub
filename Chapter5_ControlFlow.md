# Chapter 5: Control Flow

## 5.1 Introduction

two primary tools of control flow: choices and loops

**choices:** allow you to run different code depending on the input (ex,
`if` statements and `switch()`)

**loops:** allow you to repeatedly run code, typically with changing
options (ex, for and while)

## 5.2 Choices

basic form of an if statement in R:

    #if (condition) true_action
    #if (condition) true_action else false_action

If the condition is true, the true action is evaluated. If the condition
if false, the false action is evaluated.

Typically the actions are compound statements contained within {:

    grade <- function(x) {
      if (x > 90) {
        "A"
      } else if (x > 80) {
        "B"
      } else if (x > 50) {
        "C"
      } else {
        "F"
      }
    }

if returns a value so that you can assign the results:

    x1 <- if (TRUE) 1 else 2
    x2 <- if (FALSE) 1 else 2

    c(x1, x2)

    ## [1] 1 2

    #> [1] 1 2

When you use the single argument form without an else statement, if
invisibly (Section 6.7.2) returns NULL if the condition is FALSE. Since
functions like c() and paste() drop NULL inputs, this allows for a
compact expression of certain idioms:

    greet <- function(name, birthday = FALSE) {
      paste0(
        "Hi ", name,
        if (birthday) " and HAPPY BIRTHDAY"
      )
    }
    greet("Maria", FALSE)

    ## [1] "Hi Maria"

    #> [1] "Hi Maria"
    greet("Jaime", TRUE)

    ## [1] "Hi Jaime and HAPPY BIRTHDAY"

    #> [1] "Hi Jaime and HAPPY BIRTHDAY"

## 5.2.1 Invalid Inputs

The condition should evaluate to a single TRUE or FALSE. Most other
inputs will generate an error:

    #if ("x") 1
    #> Error in if ("x") 1: argument is not interpretable as logical
    #if (logical()) 1
    #> Error in if (logical()) 1: argument is of length zero
    #if (NA) 1
    #> Error in if (NA) 1: missing value where TRUE/FALSE needed

The exception is a logical vector of length greater than 1, which
generates a warning (but it IS a mistake/error):

    if (c(TRUE, FALSE)) 1

    ## Warning in if (c(TRUE, FALSE)) 1: the condition has length > 1 and only the
    ## first element will be used

    ## [1] 1

    #> Warning in if (c(TRUE, FALSE)) 1: the condition has length > 1 and only the
    #> first element will be used
    #> [1] 1

## 5.2.2 Vectorized if

Given that if only works with a single TRUE or FALSE, you might wonder
what to do if you have a vector of logical values. Handling vectors of
values is the job of ifelse(): a vectorised function with test, yes, and
no vectors (that will be recycled to the same length):

    x <- 1:10
    ifelse(x %% 5 == 0, "XXX", as.character(x))

    ##  [1] "1"   "2"   "3"   "4"   "XXX" "6"   "7"   "8"   "9"   "XXX"

    #>  [1] "1"   "2"   "3"   "4"   "XXX" "6"   "7"   "8"   "9"   "XXX"

    ifelse(x %% 2 == 0, "even", "odd")

    ##  [1] "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even"

    #>  [1] "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even"

Note that missing values will be propagated into the output.

I recommend using ifelse() only when the yes and no vectors are the same
type as it is otherwise hard to predict the output type.

Another vectorised equivalent is the more general dplyr::case\_when().
It uses a special syntax to allow any number of condition-vector pairs:

    dplyr::case_when(
      x %% 35 == 0 ~ "fizz buzz",
      x %% 5 == 0 ~ "fizz",
      x %% 7 == 0 ~ "buzz",
      is.na(x) ~ "???",
      TRUE ~ as.character(x)
    )

    ##  [1] "1"    "2"    "3"    "4"    "fizz" "6"    "buzz" "8"    "9"    "fizz"

    #>  [1] "1"    "2"    "3"    "4"    "fizz" "6"    "buzz" "8"    "9"    "fizz"

## 5.2.3 switch() Statement

Closely related to if is the switch() statement. It’s a compact, special
purpose equivalent that lets you replace code like:

    x_option <- function(x) {
      if (x == "a") {
        "option 1"
      } else if (x == "b") {
        "option 2" 
      } else if (x == "c") {
        "option 3"
      } else {
        stop("Invalid `x` value")
      }
    }

with the more succinct:

    x_option <- function(x) {
      switch(x,
        a = "option 1",
        b = "option 2",
        c = "option 3",
        stop("Invalid `x` value")
      )
    }

The last component of a switch() should always throw an error, otherwise
unmatched inputs will invisibly return NULL:

    (switch("c", a = 1, b = 2))

    ## NULL

    #> NULL

If multiple inputs have the same output, you can leave the right hand
side of = empty and the input will “fall through” to the next value.
This mimics the behaviour of C’s switch statement:

    legs <- function(x) {
      switch(x,
        cow = ,
        horse = ,
        dog = 4,
        human = ,
        chicken = 2,
        plant = 0,
        stop("Unknown input")
      )
    }
    legs("cow")

    ## [1] 4

    #> [1] 4
    legs("dog")

    ## [1] 4

    #> [1] 4

It is also possible to use switch() with a numeric x, but is harder to
read, and has undesirable failure modes if x is a not a whole number. I
recommend using switch() only with character inputs.

## 5.2.4 Exercises

1.  What type of vector does each of the following calls to `ifelse()`
    return?

<!-- -->

    ifelse(TRUE, 1, "no")

    ## [1] 1

    ifelse(FALSE, 1, "no")

    ## [1] "no"

    ifelse(NA, 1, "no")

    ## [1] NA

double/integer, character, and logical vectors respectively

1.  Why does the following code work?

<!-- -->

    x <- 1:10
    if (length(x)) "not empty" else "empty"

    ## [1] "not empty"

    #> [1] "not empty"

    x <- numeric()
    if (length(x)) "not empty" else "empty"

    ## [1] "empty"

    #> [1] "empty"

**Something to do with 1:10 versus numeric()…one is empty and one is
not**

## 5.3 Loops

for loops are used to iterate over items in a vector. They have the
following basic form:

    #for (item in vector) perform_action

For each item in vector, perform\_action is called once; updating the
value of item each time.

    for (i in 1:3) {
      print(i)
    }

    ## [1] 1
    ## [1] 2
    ## [1] 3

    #> [1] 1
    #> [1] 2
    #> [1] 3

(When iterating over a vector of indices, it’s conventional to use very
short variable names like i, j, or k.)

N.B.: for assigns the item to the current environment, overwriting any
existing variable with the same name:

    i <- 100
    for (i in 1:3) {}
    i

    ## [1] 3

    #> [1] 3

There are two ways to terminate a for loop early:

next exits the current iteration. break exits the entire for loop.

    for (i in 1:10) {
      if (i < 3) 
        next

      print(i)
      
      if (i >= 5)
        break
    }

    ## [1] 3
    ## [1] 4
    ## [1] 5

    #> [1] 3
    #> [1] 4
    #> [1] 5

## 5.3.1 Common Pitfalls

There are three common pitfalls to watch out for when using for. First,
if you’re generating data, make sure to preallocate the output
container. Otherwise the loop will be very slow; see Sections 23.2.2 and
24.6 for more details. The vector() function is helpful here.

    means <- c(1, 50, 20)
    out <- vector("list", length(means))
    for (i in 1:length(means)) {
      out[[i]] <- rnorm(10, means[[i]])
    }

Next, beware of iterating over 1:length(x), which will fail in unhelpful
ways if x has length 0:

    1:length(means)

    ## [1] 1 2 3

    #> [1] 1 0

Use seq\_along(x) instead. It always returns a value the same length as
x:

    seq_along(means)

    ## [1] 1 2 3

    #> integer(0)

    out <- vector("list", length(means))
    for (i in seq_along(means)) {
      out[[i]] <- rnorm(10, means[[i]])
    }

Finally, you might encounter problems when iterating over S3 vectors, as
loops typically strip the attributes:

    xs <- as.Date(c("2020-01-01", "2010-01-01"))
    for (x in xs) {
      print(x)
    }

    ## [1] 18262
    ## [1] 14610

    #> [1] 18262
    #> [1] 14610

Work around this by calling \[\[ yourself:

    for (i in seq_along(xs)) {
      print(xs[[i]])
    }

    ## [1] "2020-01-01"
    ## [1] "2010-01-01"

    #> [1] "2020-01-01"
    #> [1] "2010-01-01"

## 5.3.2 Related Tools

two tools similar to for loops but with more flexible specifications:

\*while(condition) action: performs action while condition is TRUE.

\*repeat(action): repeats action forever (i.e. until it encounters
break).

while is more flexible than for, and repeat is more flexible than while.

## 5.3.3 Exercises

1.  Why does this code succeed without errors or warnings?

<!-- -->

    x <- numeric()
    out <- vector("list", length(x))
    for (i in 1:length(x)) {
      out[i] <- x[i] ^ 2
    }
    out

    ## [[1]]
    ## [1] NA

**The vector has a length of 0 (numeric()), and therefore as the
iterations run through, you get NA, and then NA^2, which is still just
NA…there are no mistakes in the code, it still runs okay.**

1.  When the following code is evaluated, what can you say about the
    vector being iterated?

<!-- -->

    xs <- c(1, 2, 3)
    for (x in xs) {
      xs <- c(xs, x * 2)
    }
    xs

    ## [1] 1 2 3 2 4 6

    #> [1] 1 2 3 2 4 6

The vector is an integer or double atomic vector?

**The below explanation for number 3 might be the explanation for this
one, I can’t remember.**

1.  What does the following code tell you about when the index is
    updated?

<!-- -->

    for (i in 1:3) {
      i <- i * 2
      print(i) 
    }

    ## [1] 2
    ## [1] 4
    ## [1] 6

    #> [1] 2
    #> [1] 4
    #> [1] 6

Index? I can tell that the original number (vector 1-3) is multiplied by
2, which is renamed as i. So, 1x2 = 2, 2x2 = 4, and 3x2 = 6, which are
our three outputs.

**John explanation from club meeting: The index is updated after each
iteration. 1x2 = 2, and then that output is added to the end of the
vector, which increases the vector length from 3 to 4. This continues
for each iteration. 2x2 = 4, which is added to the vector to make it 5
in length. The for loop, however, does not iterate through those added
values. It only iterates through the original values of the vector
(1-3).**

## Quiz

1.  What is the difference between `if` and `ifelse()`?

if works with scalars and ifelse() works with vectors

1.  In the following code, what will the value of y be if x is TRUE?
    What if x is FALSE? What if x is NA?

<!-- -->

    y <- if (x) 3

y = 3, y = NULL, and error

1.  What does `switch("x", x = , y = 2, z = 3)` return?

this switch statement makes use of fall-through, so it will return 2.
(section 5.2.3)
