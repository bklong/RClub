\#Chapter 7: Environments#

\##7.1 Introduction##

environment: data structure that powers scoping

prep:

    library(rlang)

\##7.2 Environment Basics##

An environment is similar to a named list, with four exceptions: -every
name must be unique -the names in an environment are not ordered -an
environment has a parent -environments are not copied when modified

\###7.2.1 Basics###

Use `rlang::env()` to make an environment. It works similarly to
`list()`, taking a set of name-value pairs:

    e1 <- env(
      a = FALSE,
      b = "a",
      c = 2.3,
      d = 1:3,
    )

Use `new.env()` to create a new environment. Ignore hash and size
parameters (not needed). You cannot simultaneously create and define
values. Use `$<-` as shown below.

The job of an environment is to associate, or bind, a set of names to a
set of values. You can think of an environment as a bag of names, with
no implied order (i.e. it doesn’t make sense to ask which is the first
element in an environment). For that reason, we’ll draw the environment
as so:

!(<https://d33wubrfki0l68.cloudfront.net/f5dbd02f5235283e78decdd4f18692b40f1ddf42/c5683/diagrams/environments/bindings.png>)

As discussed in Section 2.5.2, environments have reference semantics:
unlike most R objects, when you modify them, you modify them in place,
and don’t create a copy. One important implication is that environments
can contain themselves.

    e1$d <- e1

!(<https://d33wubrfki0l68.cloudfront.net/0d41862821d3226c38b73f78a530117349b7344a/abb88/diagrams/environments/loop.png>)

Printing an environment just displays its memory address, which is not
terribly useful:

    e1

    ## <environment: 0x000002d276357828>

    #> <environment: 0x7fe6c2184968>

Instead, we’ll use `env_print()` which gives us a little more
information:

    env_print(e1)

    ## <environment: 0x000002d276357828>
    ## Parent: <environment: global>
    ## Bindings:
    ## • a: <lgl>
    ## • b: <chr>
    ## • c: <dbl>
    ## • d: <env>

    #> <environment: 0x7fe6c2184968>
    #> parent: <environment: global>
    #> bindings:
    #>  * a: <lgl>
    #>  * b: <chr>
    #>  * c: <dbl>
    #>  * d: <env>

You can use `env_names()` to get a character vector giving the current
bindings.

    env_names(e1)

    ## [1] "a" "b" "c" "d"

    #> [1] "a" "b" "c" "d"

Use `names()` to list the bindings in an environment.

\###7.2.2 Important Environments###

`current_env()`: the environment in which code is currently executing

`global_env()`: global environment, sometimes called your “workspace”,
as it’s where all interactive computation takes place (outside of a
function)

Use `identical()` to compare environments instead of `==`, which is for
vectors.

    #identical(global_env(), current_env())
    #> [1] TRUE

    #global_env() == current_env()
    #> Error in global_env() == current_env(): comparison (1) is possible only for
    #> atomic and list types

Access the global environment with `globalenv()` and the current
environment with `environment()`. The global environment is printed as
`R_GlobalEnv` and `.GlobalEnv`.

\###7.2.3 Parents###

Every environment has a parent, another environment. In diagrams, the
parent is shown as a small pale blue circle and arrow that points to
another environment. The parent is what’s used to implement lexical
scoping: if a name is not found in an environment, then R will look in
its parent (and so on). You can set the parent environment by supplying
an unnamed argument to `env()`. If you don’t supply it, it defaults to
the current environment. In the code below, `e2a` is the parent of
`e2b`.

    e2a <- env(d = 4, e = 5)
    e2b <- env(e2a, a = 1, b = 2, c = 3)

!(<https://d33wubrfki0l68.cloudfront.net/336e61bf494a6424484b8b2685a440a7db1566bf/59bce/diagrams/environments/parents.png>)

`env_parent()` allows you to find the parent of an environment.

    env_parent(e2b)

    ## <environment: 0x000002d2778139b0>

    #> <environment: 0x7fe6c7399f58>
    env_parent(e2a)

    ## <environment: R_GlobalEnv>

    #> <environment: R_GlobalEnv>

Only one environment doesn’t have a parent: the empty environment. I
draw the empty environment with a hollow parent environment, and where
space allows I’ll label it with `R_EmptyEnv`, the name R uses.

    e2c <- env(empty_env(), d = 4, e = 5)
    e2d <- env(e2c, a = 1, b = 2, c = 3)

!(<https://d33wubrfki0l68.cloudfront.net/ff7bec1ccb1455917a6c9d0f44f114ef5c78519f/39793/diagrams/environments/parents-empty.png>)

The ancestors of every environment eventually terminate with the empty
environment. You can see all ancestors with `env_parents()`:

    env_parents(e2b)

    ## [[1]]   <env: 0x000002d2778139b0>
    ## [[2]] $ <env: global>

    #> [[1]]   <env: 0x7fe6c7399f58>
    #> [[2]] $ <env: global>
    env_parents(e2d)

    ## [[1]]   <env: 0x000002d2760d6790>
    ## [[2]] $ <env: empty>

    #> [[1]]   <env: 0x7fe6c4d9ca20>
    #> [[2]] $ <env: empty>

By default, `env_parents()` stops when it gets to the global
environment. This is useful because the ancestors of the global
environment include every attached package, which you can see if you
override the default behaviour as below.

    env_parents(e2b, last = empty_env())

    ##  [[1]]   <env: 0x000002d2778139b0>
    ##  [[2]] $ <env: global>
    ##  [[3]] $ <env: package:rlang>
    ##  [[4]] $ <env: package:stats>
    ##  [[5]] $ <env: package:graphics>
    ##  [[6]] $ <env: package:grDevices>
    ##  [[7]] $ <env: package:utils>
    ##  [[8]] $ <env: package:datasets>
    ##  [[9]] $ <env: package:methods>
    ## [[10]] $ <env: Autoloads>
    ## [[11]] $ <env: package:base>
    ## [[12]] $ <env: empty>

    #>  [[1]]   <env: 0x7fe6c7399f58>
    #>  [[2]] $ <env: global>
    #>  [[3]] $ <env: package:rlang>
    #>  [[4]] $ <env: package:stats>
    #>  [[5]] $ <env: package:graphics>
    #>  [[6]] $ <env: package:grDevices>
    #>  [[7]] $ <env: package:utils>
    #>  [[8]] $ <env: package:datasets>
    #>  [[9]] $ <env: package:methods>
    #> [[10]] $ <env: Autoloads>
    #> [[11]] $ <env: package:base>
    #> [[12]] $ <env: empty>

Use `parent.env()` to find the parent of an environment. No base
function returns all ancestors.

\###7.2.4 Super Assignment###

The ancestors of an environment have an important relationship to `<<-`.
Regular assignment, `<-`, always creates a variable in the current
environment. Super assignment, `<<-`, never creates a variable in the
current environment, but instead modifies an existing variable found in
a parent environment.

    x <- 0
    f <- function() {
      x <<- 1
    }
    f()
    x

    ## [1] 1

    #> [1] 1

If `<<-` doesn’t find an existing variable, it will create one in the
global environment. This is usually undesirable, because global
variables introduce non-obvious dependencies between functions. `<<-` is
most often used in conjunction with a function factory.

\###7.2.5 Getting and Setting###

You can get and set elements of an environment with `$` and `[[` in the
same way as a list:

    e3 <- env(x = 1, y = 2)
    e3$x

    ## [1] 1

    #> [1] 1
    e3$z <- 3
    e3[["z"]]

    ## [1] 3

    #> [1] 3

But you can’t use `[[` with numeric indices, and you can’t use `[`:

    #e3[[1]]
    #> Error in e3[[1]]: wrong arguments for subsetting an environment

    #e3[c("x", "y")]
    #> Error in e3[c("x", "y")]: object of type 'environment' is not subsettable

`$` and `[[` will return `NULL` if the binding doesn’t exist. Use
`env_get()` if you want an error:

    #e3$xyz
    #> NULL

    #env_get(e3, "xyz")
    #> Error in env_get(e3, "xyz"): argument "default" is missing, with no default

If you want to use a default value if the binding doesn’t exist, you can
use the `default` argument.

    env_get(e3, "xyz", default = NA)

    ## [1] NA

    #> [1] NA

There are two other ways to add bindings to an environment. `env_poke()`
takes a name (as string) and a value, and `env_bind()` allows you to
bind multiple values.

    env_poke(e3, "a", 100)
    e3$a

    ## [1] 100

    #> [1] 100

    env_bind(e3, a = 10, b = 20)
    env_names(e3)

    ## [1] "x" "y" "z" "a" "b"

    #> [1] "x" "y" "z" "a" "b"

You can determine if an environment has a binding with `env_has()`:

    env_has(e3, "a")

    ##    a 
    ## TRUE

    #>    a 
    #> TRUE

Unlike lists, setting an element to `NULL` does not remove it, because
sometimes you want a name that refers to `NULL`. Instead, use
`env_unbind()`:

    e3$a <- NULL
    env_has(e3, "a")

    ##    a 
    ## TRUE

    #>    a 
    #> TRUE

    env_unbind(e3, "a")
    env_has(e3, "a")

    ##     a 
    ## FALSE

    #>     a 
    #> FALSE

Unbinding a name doesn’t delete the object. That’s the job of the
garbage collector, which automatically removes objects with no names
binding to them.

See `get()`, `assign()`, `exists()`, and `rm()`. These are designed
interactively for use with the current environment, so working with
other environments is a little clunky. Also beware the `inherits`
argument: it defaults to `TRUE` meaning that the base equivalents will
inspect the supplied environment and all its ancestors.

\###7.2.6 Advanced Bindings###

two variants of `env_bind()`:

`env_bind_lazy()` creates delayed bindings, which are evaluated the
first time they are accessed. Behind the scenes, delayed bindings create
promises, so behave in the same way as function arguments.

    env_bind_lazy(current_env(), b = {Sys.sleep(1); 1})

    system.time(print(b))

    ## [1] 1

    ##    user  system elapsed 
    ##    0.00    0.00    1.03

    #> [1] 1
    #>    user  system elapsed 
    #>    0.00    0.00    1.09
    system.time(print(b))

    ## [1] 1

    ##    user  system elapsed 
    ##       0       0       0

    #> [1] 1
    #>    user  system elapsed 
    #>       0       0       0

The primary use of delayed bindings is in `autoload()`, which allows R
packages to provide datasets that behave like they are loaded in memory,
even though they’re only loaded from disk when needed.

`env_bind_active()` creates active bindings which are re-computed every
time they’re accessed.

    env_bind_active(current_env(), z1 = function(val) runif(1))

    z1

    ## [1] 0.3901937

    #> [1] 0.0808
    z1

    ## [1] 0.5891138

    #> [1] 0.834

Active bindings are used to implement R6’s active fields.

    ?delayedAssign()

    ## starting httpd help server ... done

    ?makeActiveBinding()

\###7.2.7 Exercises###

List three ways in which an environment differs from a list.

Environments have parents. Environments are not copied when modified.
The names in an environment are not ordered.

Create an environment as illustrated by this picture.

!(<https://d33wubrfki0l68.cloudfront.net/fcf3570a7ae04e6d1cc280e22b2d2822e812d6b0/3e607/diagrams/environments/recursive-1.png>)

    e1 <- env(loop = e1)

Create a pair of environments as illustrated by this picture.

!(<https://d33wubrfki0l68.cloudfront.net/8a81694cf39662e011249fead6821cc357a5ff5a/72820/diagrams/environments/recursive-2.png>)

    e2 <- env(loop = e3)
    e3 <- env(deloop = e2)

Explain why `e[[1]]` and `e[c("a", "b")]` don’t make sense when `e` is
an environment.

Environments are not vectors. The above syntax is how you would write
the code for a vector.

**Create a version of `env_poke()` that will only bind new names, never
re-bind old names. Some programming languages only do this, and are
known as single assignment languages.**

**What does this function do? How does it differ from `<<-` and why
might you prefer it?**

    #rebind <- function(name, value, env = caller_env()) {
    #  if (identical(env, empty_env())) {
     #   stop("Can't find `", name, "`", call. = FALSE)
    #  } else if (env_has(env, name)) {
    #    env_poke(env, name, value)
    #  } else {
    #    rebind(name, value, env_parent(env))
     # }
    #}
    #rebind("a", 10)
    #> Error: Can't find `a`
    #a <- 5
    #rebind("a", 10)
    #a
    #> [1] 10

\##7.3 Recursing Over Environments##

If you want to operate on every ancestor of an environment, it’s often
convenient to write a recursive function. This section shows you how,
applying your new knowledge of environments to write a function that
given a name, finds the environment `where()` that name is defined,
using R’s regular scoping rules.

The definition of `where()` is straightforward. It has two arguments:
the name to look for (as a string), and the environment in which to
start the search. (We’ll learn why `caller_env()` is a good default in
Section 7.5.)

    where <- function(name, env = caller_env()) {
      if (identical(env, empty_env())) {
        # Base case
        stop("Can't find ", name, call. = FALSE)
      } else if (env_has(env, name)) {
        # Success case
        env
      } else {
        # Recursive case
        where(name, env_parent(env))
      }
    }

three cases:

-base case: we’ve reached the empty environment and haven’t found the
binding. We can’t go any further, so we throw an error.

-successful case: the name exists in this environment, so we return the
environment.

-recursive case: the name was not found in this environment, so try the
parent.

examples:

    #where("yyy")
    #> Error: Can't find yyy

    #x <- 5
    #where("x")
    #> <environment: R_GlobalEnv>

    #where("mean")
    #> <environment: base>

It might help to see a picture. Imagine you have two environments, as in
the following code and diagram:

    e4a <- env(empty_env(), a = 1, b = 2)
    e4b <- env(e4a, x = 10, a = 11)

!(<https://d33wubrfki0l68.cloudfront.net/9fab27eb096eb643a391f207daeabbb023813c30/7e894/diagrams/environments/where-ex.png>)

`where("a", e4b)` will find `a` in `e4b`.

`where("b", e4b)` doesn’t find `b` in `e4b`, so it looks in its parent,
`e4a`, and finds it there.

`where("c", e4b)` looks in `e4b`, then `e4a`, then hits the empty
environment and throws an error.

It’s natural to work with environments recursively, so `where()`
provides a useful template. Removing the specifics of `where()` shows
the structure more clearly:

    f <- function(..., env = caller_env()) {
      if (identical(env, empty_env())) {
        # base case
      } else if (success) {
        # success case
      } else {
        # recursive case
        f(..., env = env_parent(env))
      }
    }

\####Iteration Versus Recursion####

It’s possible to use a loop instead of recursion. I think it’s harder to
understand than the recursive version, but I include it because you
might find it easier to see what’s happening if you haven’t written many
recursive functions.

    f2 <- function(..., env = caller_env()) {
      while (!identical(env, empty_env())) {
        if (success) {
          # success case
          return()
        }
        # inspect parent
        env <- env_parent(env)
      }

      # base case
    }

\###7.3.1 Exercises###

Modify `where()` to return all environments that contain a binding for
`name`. Carefully think through what type of object the function will
need to return.

    #for loop?

**Write a function called `fget()` that finds only function objects. It
should have two arguments, `name` and `env`, and should obey the regular
scoping rules for functions: if there’s an object with a matching name
that’s not a function, look in the parent. For an added challenge, also
add an `inherits` argument which controls whether the function recurses
up the parents or only looks in one environment.**

\##7.4 Special Environments##

\##Quiz##

List at least three ways that an environment differs from a list.

What is the parent of the global environment? What is the only
environment that doesn’t have a parent?

What is the enclosing environment of a function? Why is it important?

How do you determine the environment from which a function was called?

How are &lt;- and &lt;&lt;- different?
