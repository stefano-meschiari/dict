`dict` provides a new data structure specialized for representing dictionaries with string keys.

Install the package using [devtools](http://cran.r-%20project.org/package=devtools):

``` r
devtools::install_github('stefano-meschiari/dict')
```

and import the library with `library`:

``` r
library(dict)
```

A proper dictionary class
=========================

The new class `dict` can be used to represent a heterogeneous dictionary with character keys. `dict`s have a few advantages over named lists:

-   Every value is uniquely associated with a key (no holes).
-   Keys are unique and cannot be repeated.
-   Printing of dicts is more compact.
-   Keys are never partially matched (a key named `test` will not match with `t` or `te`).
-   A dict can have default values for non-existing keys.
-   Dicts can contain NULL values.

This library also provides some useful functions for manipulating dictionary-like objects.

Creating a dictionary
---------------------

The `dict` function can be used to create a new dictionary:

``` r
d <- dict(color='blue', pattern='solid', width=3)
```

You can create a dictionary out of a list of keys and values using `make_dict`:

``` r
d <- make_dict(keys=c('color', 'pattern', 'width'),
               values=c('blue', 'solid', 3))
```

You can convert a named list or vector to a dictionary using `as_dict`:

``` r
d <- as_dict(list(color='blue', pattern='solid', width=3))
d <- as_dict(c("a" = 1, "b" = 2))
```

Printing looks nice:

``` r
print(d)
```

    ##   $ color : [1] "blue"
    ## $ pattern : [1] "solid"
    ##   $ width : [1] 3

You can use a shorthand and leave keys implicit (this is similar to the ES6 object literal shorthand). `dict` will try to create implicit keys based on the arguments:

``` r
operating_system <- 'Amiga OS'
version <- '3.1'
cpu <- '68040'

machine <- dict(operating_system, version, cpu)

print(machine)
```

    ## $ operating_system : [1] "Amiga OS"
    ##          $ version : [1] "3.1"
    ##              $ cpu : [1] "68040"

Accessing entries
-----------------

You can access values using the familiar R syntax for lists:

``` r
d <- dict(color='blue', pattern='solid', width=3)

d$color                   # blue
d[['pattern']]            # solid
d['color', 'pattern']  # a sub-dictionary with keys color and pattern
```

You can get keys and values using the `keys` and `values` functions:

``` r
keys(d)      # c('color', 'pattern', 'width')
values(d)    # list('blue', 'solid', 3)
```

You can get a list of "entries" (each element being a list with `key` and `value`). This is useful for iteration:

``` r
for (entry in entries(d)) {
  cat(entry$key, ' = ', entry$value)
}
```

You can use the `map_dict` function to map over keys and values, and the `keep_dict` and `discard_dict` to filter entries. These functions are specializations of the `map`, `keep` and `discard` family of functions in [purrr](https://github.com/hadley/purrr) to the dictionary class.

``` r
# Returns a dict with the same keys and squared values
d <- dict(a=1, b=2, c=3)
map_dict(d, function(k, v) v^2)
```

Setting entries
---------------

Entries can be set just like regular lists:

``` r
d <- dict(first='Harrison', last='Solo')  # first=Harrison, last=Solo
d$last <- 'Ford'                          # first=Harrison, last=Ford
d[['first']] <- 'Leia'                    # first=Leia, last=Ford
d[c('last', 'title')] <- c('Organa', 'Princess') # first=Leia, last=Organa, title=Princess
```

Setting an entry to `NULL` does *not* delete the entry, but instead sets the entry to `NULL`. To delete one or more entries, use the `omit` function:

``` r
d <- dict(a=1, c=3)
d$b <- NULL         # a=1, b=NULL, c=3
d <- omit(a, 'a')   # b=NULL, c=3
```

Utility functions
-----------------

A few utility functions inspired by the Underscore library:

-   `invert(dict)` returns a dictionary where keys and values are swapped.
-   `has(dict, key)` returns TRUE if `dict` contains `key`.
-   `omit(dict, key1, key2, ...)` returns a new dictionary omitting all the specified keys.
-   `extend(dict, dict1, ...)` copies all entries in `dict1` into `dict`, overriding any existing entries and returns a new dictionary.
-   `defaults(dict, defaults)` fill in entries from `defaults` into `dict` for any keys missing in `dict`.

The following functions specialize functions from the `purrr` library to dictionary objects:

-   `map_dict(dict, fun, ...)` calls a function on each (key, value) pair and builds a dictionary from the transformed input.
-   `keep_dict(dict, fun)` and `discard_dict(dict, fun)` keep or discard entries based on the function or predicate.
-   `compact_dict(dict)` removes any entries with NULL values.

Default dictionaries
--------------------

You can create a dictionary with default values using `default_dict`. Any time a non-existing key is accessed, the default value is returned.

``` r
salaries <- default_dict(employee_a = 50000,
                         employee_b = 100000,
                         default = 65000)
```

You can provide a default value for an existing dictionary by setting its `default` attribute:

``` r
attr(salaries, 'default') <- 70000
```

Strict dictionaries
-------------------

You can create a strict dictionary using `strict_dict`. Any time a non-existing key is accessed, an exception is thrown using `stop()`.

``` r
# Associating each letter with a number
letters_to_numbers <- strict_dict(a=1, b=2, c=3, d=4) # etc.

# Accessing an existing key, that's OK!
print(letters_to_numbers$a)
```

    ## [1] 1

``` r
# Accessing a non-letter will throw!
tryCatch(letters_to_numbers$notaletter, error=function(e) print(e))
```

    ## <simpleError in `$.dict`(letters_to_numbers, notaletter): Attempted access of non-existing keynotaletter>

Immutable collections
---------------------

You can turn any collection (lists, vectors, and `dicts`) into an immutable collection using the `immutable` function. Such a collection cannot be modified using the `[`, `[[` and `$` operators; otherwise, it will behave the same as the original collection. The `immutable_dict` function creates an immutable dictionary.

``` r
const_letters <- immutable(letters)

# Will throw an error
const_letters[1] <- '1' 
```

    ## Error in `[<-.immutable`(`*tmp*`, 1, value = "1"): Attempting to mutate an immutable collection.

``` r
physical_constants <- immutable_dict(
  astronomical_unit = 1.5e11,
  speed_of_light = 3e8,
  gravitational_constant = 6.7e-11
)

# Will throw an error
physical_constants$speed_of_light = 1
```

    ## Error in `$<-.immutable`(`*tmp*`, "speed_of_light", value = 1): Attempting to mutate an immutable collection.
