# Chapter 8: Conditions

## 8.1 Introduction

Conditions give the coder a set of tools to indicated something unusual
is happening with a function. The function author can signal conditions
with functions.

Two books that are good references: “A prototype of a condition system
for R” and “Beyond exception handling: conditions and restarts”

### 8.1.1 Prereqs

We will need R base functions and condition signalling and handling
functions from rlang.

    library(rlang)

### 8.2 Signalling Conditions

There are three conditions that you can signal in code: errors,
warnings, and messages.

Errors: most severe, indicate that there is no way for a function to
continue and execution must stop.

Warnings: fall somewhere between errors and message, typically indicate
something has gone wrong but the function has been able to at least
partially recover.

Messages: mildest, a way of informing users that some actions has been
performed on their behalf.

There is a final condition that can only be generated interactively: an
interrupt, which indicates that the user has interrupted execution by
pressing Escape, Ctrl + Break, or Ctrl + C (depending on the platform).

Conditions are usually displayed prominently, in a bold font or coloured
red, depending on the R interface. You can tell them apart because
errors always start with “Error”, warnings with “Warning” or “Warning
message”, and messages with nothing.

    #stop("This is what an error looks like")
    #> Error in eval(expr, envir, enclos): This is what an error looks like

    warning("This is what a warning looks like")

    ## Warning: This is what a warning looks like

    #> Warning: This is what a warning looks like

    message("This is what a message looks like")

    ## This is what a message looks like

    #> This is what a message looks like

### 8.2.1 Errors

In base R, errors are signalled, or thrown, by stop():

    #f <- function() g()
    #g <- function() h()
    #h <- function() stop("This is an error!")

    #f()
    #> Error in h(): This is an error!

By default, the error message includes the call, but this is typically
not useful (and recapitulates information that you can easily get from
traceback()), so I think it’s good practice to use call. = FALSE46:

    #h <- function() stop("This is an error!", call. = FALSE)
    #f()
    #> Error: This is an error!

(NB: stop() pastes together multiple inputs, while abort() does not. To
create complex error messages with abort, I recommend using
glue::glue(). This allows us to use other arguments to abort() for
useful features that you’ll learn about in Section 8.5.)

The best error messages tell you what is wrong and point you in the
right direction to fix the problem. Writing good error messages is hard
because errors usually occur when the user has a flawed mental model of
the function. As a developer, it’s hard to imagine how the user might be
thinking incorrectly about your function, and thus it’s hard to write a
message that will steer the user in the correct direction. That said,
the tidyverse style guide discusses a few general principles that we
have found useful: <http://style.tidyverse.org/error-messages.html>.

### 8.2.2 Warnings

Warnings, signalled by warning(), are weaker than errors: they signal
that something has gone wrong, but the code has been able to recover and
continue. Unlike errors, you can have multiple warnings from a single
function call:

    fw <- function() {
      cat("1\n")
      warning("W1")
      cat("2\n")
      warning("W2")
      cat("3\n")
      warning("W3")
    }

By default, warnings are cached and printed only when control returns to
the top level:

    fw()

    ## 1

    ## Warning in fw(): W1

    ## 2

    ## Warning in fw(): W2

    ## 3

    ## Warning in fw(): W3

    #> 1
    #> 2
    #> 3
    #> Warning messages:
    #> 1: In f() : W1
    #> 2: In f() : W2
    #> 3: In f() : W3

You can control this behaviour with the warn option:

To make warnings appear immediately, set options(warn = 1).

To turn warnings into errors, set options(warn = 2). This is usually the
easiest way to debug a warning, as once it’s an error you can use tools
like traceback() to find the source.

Restore the default behaviour with options(warn = 0).

Like stop(), warning() also has a call argument. It is slightly more
useful (since warnings are often more distant from their source), but I
still generally suppress it with call. = FALSE. Like rlang::abort(), the
rlang equivalent of warning(), rlang::warn(), also suppresses the call.
by default.

Warnings occupy a somewhat challenging place between messages (“you
should know about this”) and errors (“you must fix this!”), and it’s
hard to give precise advice on when to use them. Generally, be
restrained, as warnings are easy to miss if there’s a lot of other
output, and you don’t want your function to recover too easily from
clearly invalid input. In my opinion, base R tends to overuse warnings,
and many warnings in base R would be better off as errors. For example,
I think these warnings would be more helpful as errors:

    formals(1)

    ## Warning in formals(fun): argument is not a function

    ## NULL

    #> Warning in formals(fun): argument is not a function
    #> NULL

    file.remove("this-file-doesn't-exist")

    ## Warning in file.remove("this-file-doesn't-exist"): cannot remove file 'this-
    ## file-doesn't-exist', reason 'No such file or directory'

    ## [1] FALSE

    #> Warning in file.remove("this-file-doesn't-exist"): cannot remove file 'this-
    #> file-doesn't-exist', reason 'No such file or directory'
    #> [1] FALSE

    lag(1:3, k = 1.5)

    ## Warning in lag.default(1:3, k = 1.5): 'k' is not an integer

    ## [1] 1 2 3
    ## attr(,"tsp")
    ## [1] -1  1  1

    #> Warning in lag.default(1:3, k = 1.5): 'k' is not an integer
    #> [1] 1 2 3
    #> attr(,"tsp")
    #> [1] -1  1  1

    as.numeric(c("18", "30", "50+", "345,678"))

    ## Warning: NAs introduced by coercion

    ## [1] 18 30 NA NA

    #> Warning: NAs introduced by coercion
    #> [1] 18 30 NA NA

There are only a couple of cases where using a warning is clearly
appropriate:

When you deprecate a function you want to allow older code to continue
to work (so ignoring the warning is OK) but you want to encourage the
user to switch to a new function.

When you are reasonably certain you can recover from a problem: If you
were 100% certain that you could fix the problem, you wouldn’t need any
message; if you were more uncertain that you could correctly fix the
issue, you’d throw an error.

Otherwise use warnings with restraint, and carefully consider if an
error would be more appropriate.

### 8.2.3 Messages

Messages, signalled by message(), are informational; use them to tell
the user that you’ve done something on their behalf. Good messages are a
balancing act: you want to provide just enough information so the user
knows what’s going on, but not so much that they’re overwhelmed.

message()s are displayed immediately and do not have a call. argument:

    fm <- function() {
      cat("1\n")
      message("M1")
      cat("2\n")
      message("M2")
      cat("3\n")
      message("M3")
    }

    fm()

    ## 1

    ## M1

    ## 2

    ## M2

    ## 3

    ## M3

    #> 1
    #> M1
    #> 2
    #> M2
    #> 3
    #> M3

Good places to use a message are:

When a default argument requires some non-trivial amount of computation
and you want to tell the user what value was used. For example, ggplot2
reports the number of bins used if you don’t supply a binwidth.

In functions that are called primarily for their side-effects which
would otherwise be silent. For example, when writing files to disk,
calling a web API, or writing to a database, it’s useful to provide
regular status messages telling the user what’s happening.

When you’re about to start a long running process with no intermediate
output. A progress bar (e.g. with progress) is better, but a message is
a good place to start.

When writing a package, you sometimes want to display a message when
your package is loaded (i.e. in .onAttach()); here you must use
packageStartupMessage().

Generally any function that produces a message should have some way to
suppress it, like a quiet = TRUE argument. It is possible to suppress
all messages with suppressMessages(), as you’ll learn shortly, but it is
nice to also give finer grained control.

It’s important to compare message() to the closely related cat(). In
terms of usage and result, they appear quite similar47:

    cat("Hi!\n")

    ## Hi!

    #> Hi!

    message("Hi!")

    ## Hi!

    #> Hi!

However, the purposes of cat() and message() are different. Use cat()
when the primary role of the function is to print to the console, like
print() or str() methods. Use message() as a side-channel to print to
the console when the primary purpose of the function is something else.
In other words, cat() is for when the user asks for something to be
printed and message() is for when the developer elects to print
something.

### 8.2.4 Exercises

1.  Write a wrapper around file.remove() that throws an error if the
    file to be deleted does not exist.

2.  What does the appendLF argument to message() do? How is it related
    to cat()?

## 8.3 Ignoring Conditions

The simplest way of handling conditions in R is to simply ignore them:

Ignore errors with try(). Ignore warnings with suppressWarnings().
Ignore messages with suppressMessages(). These functions are heavy
handed as you can’t use them to suppress a single type of condition that
you know about, while allowing everything else to pass through. We’ll
come back to that challenge later in the chapter.

try() allows execution to continue even after an error has occurred.
Normally if you run a function that throws an error, it terminates
immediately and doesn’t return a value:

    #f1 <- function(x) {
    #  log(x)
    #  10
    #}
    #f1("x")
    #> Error in log(x): non-numeric argument to mathematical function

However, if you wrap the statement that creates the error in try(), the
error message will be displayed48 but execution will continue:

    f2 <- function(x) {
      try(log(x))
      10
    }
    f2("a")

    ## Error in log(x) : non-numeric argument to mathematical function

    ## [1] 10

    #> Error in log(x) : non-numeric argument to mathematical function
    #> [1] 10

It is possible, but not recommended, to save the result of try() and
perform different actions based on whether or not the code succeeded or
failed49. Instead, it is better to use tryCatch() or a higher-level
helper; you’ll learn about those shortly.

A simple, but useful, pattern is to do assignment inside the call: this
lets you define a default value to be used if the code does not succeed.
This works because the argument is evaluated in the calling environment,
not inside the function. (See Section 6.5.1 for more details.)

    default <- NULL
    try(default <- read.csv("possibly-bad-input.csv"), silent = TRUE)

    ## Warning in file(file, "rt"): cannot open file 'possibly-bad-input.csv': No such
    ## file or directory

suppressWarnings() and suppressMessages() suppress all warnings and
messages. Unlike errors, messages and warnings don’t terminate
execution, so there may be multiple warnings and messages signalled in a
single block.

    suppressWarnings({
      warning("Uhoh!")
      warning("Another warning")
      1
    })

    ## [1] 1

    #> [1] 1

    suppressMessages({
      message("Hello there")
      2
    })

    ## [1] 2

    #> [1] 2

    suppressWarnings({
      message("You can still see me")
      3
    })

    ## You can still see me

    ## [1] 3

    #> You can still see me
    #> [1] 3

## 8.4 Handling Conditions

Every condition has default behaviour: errors stop execution and return
to the top level, warnings are captured and displayed in aggregate, and
messages are immediately displayed. Condition handlers allow us to
temporarily override or supplement the default behaviour.

Two functions, tryCatch() and withCallingHandlers(), allow us to
register handlers, functions that take the signalled condition as their
single argument. The registration functions have the same basic form:

    #tryCatch(
    #  error = function(cnd) {
        # code to run when error is thrown
    #  },
    #  code_to_run_while_handlers_are_active
    #)

    #withCallingHandlers(
    #  warning = function(cnd) {
        # code to run when warning is signalled
    #  },
    #  message = function(cnd) {
        # code to run when message is signalled
    #  },
    #  code_to_run_while_handlers_are_active
    #)

They differ in the type of handlers that they create:

tryCatch() defines exiting handlers; after the condition is handled,
control returns to the context where tryCatch() was called. This makes
tryCatch() most suitable for working with errors and interrupts, as
these have to exit anyway.

withCallingHandlers() defines calling handlers; after the condition is
captured control returns to the context where the condition was
signalled. This makes it most suitable for working with non-error
conditions.

But before we can learn about and use these handlers, we need to talk a
little bit about condition objects. These are created implicitly
whenever you signal a condition, but become explicit inside the handler.

### 8.4.1 Condition Objects

So far we’ve just signalled conditions, and not looked at the objects
that are created behind the scenes. The easiest way to see a condition
object is to catch one from a signalled condition. That’s the job of
rlang::catch\_cnd():

    cnd <- catch_cnd(stop("An error"))
    str(cnd)

    ## List of 2
    ##  $ message: chr "An error"
    ##  $ call   : language force(expr)
    ##  - attr(*, "class")= chr [1:3] "simpleError" "error" "condition"

    #> List of 2
    #>  $ message: chr "An error"
    #>  $ call   : language force(expr)
    #>  - attr(*, "class")= chr [1:3] "simpleError" "error" "condition"

Built-in conditions are lists with two elements:

message, a length-1 character vector containing the text to display to a
user. To extract the message, use conditionMessage(cnd).

call, the call which triggered the condition. As described above, we
don’t use the call, so it will often be NULL. To extract it, use
conditionCall(cnd).

Custom conditions may contain other components, which we’ll discuss in
Section 8.5.

Conditions also have a class attribute, which makes them S3 objects. We
won’t discuss S3 until Chapter 13, but fortunately, even if you don’t
know about S3, condition objects are quite simple. The most important
thing to know is that the class attribute is a character vector, and it
determines which handlers will match the condition.

### 8.4.2 Exiting Handlers

tryCatch() registers exiting handlers, and is typically used to handle
error conditions. It allows you to override the default error behaviour.
For example, the following code will return NA instead of throwing an
error:

    f3 <- function(x) {
      tryCatch(
        error = function(cnd) NA,
        log(x)
      )
    }

    f3("x")

    ## [1] NA

    #> [1] NA

If no conditions are signalled, or the class of the signalled condition
does not match the handler name, the code executes normally:

    tryCatch(
      error = function(cnd) 10,
      1 + 1
    )

    ## [1] 2

    #> [1] 2

    tryCatch(
      error = function(cnd) 10,
      {
        message("Hi!")
        1 + 1
      }
    )

    ## Hi!

    ## [1] 2

    #> Hi!
    #> [1] 2

The handlers set up by tryCatch() are called exiting handlers because
after the condition is signalled, control passes to the handler and
never returns to the original code, effectively meaning that the code
exits:

    tryCatch(
      message = function(cnd) "There",
      {
        message("Here")
        stop("This code is never run!")
      }
    )

    ## [1] "There"

    #> [1] "There"

The protected code is evaluated in the environment of tryCatch(), but
the handler code is not, because the handlers are functions. This is
important to remember if you’re trying to modify objects in the parent
environment.

The handler functions are called with a single argument, the condition
object. I call this argument cnd, by convention. This value is only
moderately useful for the base conditions because they contain
relatively little data. It’s more useful when you make your own custom
conditions, as you’ll see shortly.

    tryCatch(
      error = function(cnd) {
        paste0("--", conditionMessage(cnd), "--")
      },
      stop("This is an error")
    )

    ## [1] "--This is an error--"

    #> [1] "--This is an error--"

tryCatch() has one other argument: finally. It specifies a block of code
(not a function) to run regardless of whether the initial expression
succeeds or fails. This can be useful for clean up, like deleting files,
or closing connections. This is functionally equivalent to using
on.exit() (and indeed that’s how it’s implemented) but it can wrap
smaller chunks of code than an entire function.

    path <- tempfile()
    tryCatch(
      {
        writeLines("Hi!", path)
        # ...
      },
      finally = {
        # always run
        unlink(path)
      }
    )

### 8.4.3 Calling Handlers

The handlers set up by tryCatch() are called exiting handlers, because
they cause code to exit once the condition has been caught. By contrast,
withCallingHandlers() sets up calling handlers: code execution continues
normally once the handler returns. This tends to make
withCallingHandlers() a more natural pairing with the non-error
conditions. Exiting and calling handlers use “handler” in slighty
different senses:

An exiting handler handles a signal like you handle a problem; it makes
the problem go away.

A calling handler handles a signal like you handle a car; the car still
exists.

Compare the results of tryCatch() and withCallingHandlers() in the
example below. The messages are not printed in the first case, because
the code is terminated once the exiting handler completes. They are
printed in the second case, because a calling handler does not exit.

    tryCatch(
      message = function(cnd) cat("Caught a message!\n"), 
      {
        message("Someone there?")
        message("Why, yes!")
      }
    )

    ## Caught a message!

    #> Caught a message!

    withCallingHandlers(
      message = function(cnd) cat("Caught a message!\n"), 
      {
        message("Someone there?")
        message("Why, yes!")
      }
    )

    ## Caught a message!

    ## Someone there?

    ## Caught a message!

    ## Why, yes!

    #> Caught a message!
    #> Someone there?
    #> Caught a message!
    #> Why, yes!

Handlers are applied in order, so you don’t need to worry about getting
caught in an infinite loop. In the following example, the message()
signalled by the handler doesn’t also get caught:

    withCallingHandlers(
      message = function(cnd) message("Second message"),
      message("First message")
    )

    ## Second message

    ## First message

    #> Second message
    #> First message

(But beware if you have multiple handlers, and some handlers signal
conditions that could be captured by another handler: you’ll need to
think through the order carefully.)

The return value of a calling handler is ignored because the code
continues to execute after the handler completes; where would the return
value go? That means that calling handlers are only useful for their
side-effects.

One important side-effect unique to calling handlers is the ability to
muffle the signal. By default, a condition will continue to propagate to
parent handlers, all the way up to the default handler (or an exiting
handler, if provided):

    # Bubbles all the way up to default handler which generates the message
    withCallingHandlers(
      message = function(cnd) cat("Level 2\n"),
      withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
      )
    )

    ## Level 1
    ## Level 2

    ## Hello

    #> Level 1
    #> Level 2
    #> Hello

    # Bubbles up to tryCatch
    tryCatch(
      message = function(cnd) cat("Level 2\n"),
      withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
      )
    )

    ## Level 1
    ## Level 2

    #> Level 1
    #> Level 2

If you want to prevent the condition “bubbling up” but still run the
rest of the code in the block, you need to explicitly muffle it with
rlang::cnd\_muffle():

    # Muffles the default handler which prints the messages
    withCallingHandlers(
      message = function(cnd) {
        cat("Level 2\n")
        cnd_muffle(cnd)
      },
      withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
      )
    )

    ## Level 1
    ## Level 2

    #> Level 1
    #> Level 2

    # Muffles level 2 handler and the default handler
    withCallingHandlers(
      message = function(cnd) cat("Level 2\n"),
      withCallingHandlers(
        message = function(cnd) {
          cat("Level 1\n")
          cnd_muffle(cnd)
        },
        message("Hello")
      )
    )

    ## Level 1

    #> Level 1

### 8.4.4 Call Stacks

To complete the section, there are some important differences between
the call stacks of exiting and calling handlers. These differences are
generally not important but I’m including them here because I’ve
occasionally found them useful, and don’t want to forget about them!

It’s easiest to see the difference by setting up a small example that
uses lobstr::cst():

    f <- function() g()
    g <- function() h()
    h <- function() message("!")

Calling handlers are called in the context of the call that signalled
the condition:

    withCallingHandlers(f(), message = function(cnd) {
      lobstr::cst()
      cnd_muffle(cnd)
    })

    ##      ▆
    ##   1. ├─base::withCallingHandlers(...)
    ##   2. ├─global f()
    ##   3. │ └─global g()
    ##   4. │   └─global h()
    ##   5. │     └─base::message("!")
    ##   6. │       ├─base::withRestarts(...)
    ##   7. │       │ └─base (local) withOneRestart(expr, restarts[[1L]])
    ##   8. │       │   └─base (local) doWithOneRestart(return(expr), restart)
    ##   9. │       └─base::signalCondition(cond)
    ##  10. └─global `<fn>`(`<smplMssg>`)
    ##  11.   └─lobstr::cst()

    #>      █
    #>   1. ├─base::withCallingHandlers(...)
    #>   2. ├─global::f()
    #>   3. │ └─global::g()
    #>   4. │   └─global::h()
    #>   5. │     └─base::message("!")
    #>   6. │       ├─base::withRestarts(...)
    #>   7. │       │ └─base:::withOneRestart(expr, restarts[[1L]])
    #>   8. │       │   └─base:::doWithOneRestart(return(expr), restart)
    #>   9. │       └─base::signalCondition(cond)
    #>  10. └─(function (cnd) ...
    #>  11.   └─lobstr::cst()

Whereas exiting handlers are called in the context of the call to
tryCatch():

    tryCatch(f(), message = function(cnd) lobstr::cst())

    ##     ▆
    ##  1. └─base::tryCatch(f(), message = function(cnd) lobstr::cst())
    ##  2.   └─base (local) tryCatchList(expr, classes, parentenv, handlers)
    ##  3.     └─base (local) tryCatchOne(expr, names, parentenv, handlers[[1L]])
    ##  4.       └─value[[3L]](cond)
    ##  5.         └─lobstr::cst()

    #>     █
    #>  1. └─base::tryCatch(f(), message = function(cnd) lobstr::cst())
    #>  2.   └─base:::tryCatchList(expr, classes, parentenv, handlers)
    #>  3.     └─base:::tryCatchOne(expr, names, parentenv, handlers[[1L]])
    #>  4.       └─value[[3L]](cond)
    #>  5.         └─lobstr::cst()

### 8.4.5 Exercises

1.  What extra information does the condition generated by abort()
    contain compared to the condition generated by stop() i.e. what’s
    the difference between these two objects? Read the help for ?abort
    to learn more.

<!-- -->

    catch_cnd(stop("An error"))

    ## <simpleError in force(expr): An error>

    catch_cnd(abort("An error"))

    ## <error/rlang_error>
    ## Error:
    ## ! An error
    ## ---
    ## Backtrace:

1.  Predict the results of evaluating the following code:

<!-- -->

    show_condition <- function(code) {
      tryCatch(
        error = function(cnd) "error",
        warning = function(cnd) "warning",
        message = function(cnd) "message",
        {
          code
          NULL
        }
      )
    }

    show_condition(stop("!"))

    ## [1] "error"

    show_condition(10)

    ## NULL

    show_condition(warning("?!"))

    ## [1] "warning"

    show_condition({
      10
      message("?")
      warning("?!")
    })

    ## [1] "message"

1.  Explain the results of running this code:

<!-- -->

    withCallingHandlers(
      message = function(cnd) message("b"),
      withCallingHandlers(
        message = function(cnd) message("a"),
        message("c")
      )
    )

    ## b

    ## a

    ## b

    ## c

    #> b
    #> a
    #> b
    #> c

1.  Read the source code for catch\_cnd() and explain how it works.

2.  How would you rewrite show\_condition() to use a single handler?

## Quiz

1.  What are the three most important types of condition?
2.  What function do you use to ignore errors in block of code?
3.  What’s the main difference between tryCatch() and
    withCallingHandlers()?
4.  Why might you want to create a custom error object?
