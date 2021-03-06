
<!-- README.md is generated from README.Rmd. Please edit that file -->

# rd

<!-- badges: start -->
<!-- badges: end -->

**Quick and Easy way to read multiple files from a directory.**

## Installation

You can install rd from [GitHub](https://github.com/Lightbridge-AI/rd)
with:

``` r
# install.packages("devtools")
devtools::install_github("Lightbridge-AI/rd")
```

## How to read multiple files from a directory

Usually, if you want to read any multiple files form a directory
(folder) into a list you can do this …

**Step 1 : list all files path with `list.files()`**

``` r
abs.path <- list.files("path/to/dir", full.names = T)
```

**Step 2 : Filter files you want**

``` r
csv.abs.path <- grep("\\.csv$", abs.path, value = T) # select only .csv files
```

**Step 3 : Loop each file path to read function** (use `utils::read.csv`
in this case )

``` r
list_of_df <- lapply(csv.abs.path, utils::read.csv) # or you can use `purrr:map()`
```

**Step 4 : Set files name to each component (data frame) of the list**

``` r
names(list_of_df) <- sub("\\.[^\\.]+$","", basename(csv.abs.path))

list_of_df
```

`basename()` : gives file names

`sub("\\.[^\\.]+$","", basename(csv.abs.path))` : removes everything
after the last dot

## Quick & Easy Way !

`rd` package wrap all of the above code in to a single function: `rd()`

`rd()` has 3 main arguments

1.  `.f` : Function to read files from a directory (any function that
    has file path as first argument will work)

2.  `path` : Path to desired directory

3.  `pattern` : Regular expression to match file names and/or file
    extensions

### Examples

-   **Read all .csv files form working directory** (default) using
    `utils::read.csv` .
-   File names are automatically set to names of each data frame.

``` r
library(rd)

rd(utils::read.csv) # default `pattern` is "\\.csv$" (csv files)
```

-   **Read .xlsx file from specified directory** using
    `readxl::read_excel` .
-   Must specify regular expression to match file extension.

``` r
rd(readxl::read_excel, path = "path/to/dir" ,pattern = "\\.xlsx$") 
```

-   **You can pass extra arguments to `.f` by 2 ways**

    1.  Pass inside `.f` : using formula style similar to
        [`purrr`](https://purrr.tidyverse.org/reference/map.html)
        package (recommend)

    2.  Pass outside `.f` : an argument `...` of `rd()` will passed to
        `.f`

``` r
# Pass `col_names = FALSE` to `readr::read_csv`
rd(~readr::read_csv(.x, col_names = FALSE), recursive = TRUE, snake_case = TRUE) # inside `.f`

rd(readr::read_csv, recursive = TRUE, snake_case = TRUE, col_names = FALSE) # outside `.f`
```

Passed an argument `col_names = FALSE` into `readr::read_csv`

`recursive = TRUE` to also read files from sub-directory

`snake_case = TRUE` to format names to snake\_case (require `snakecase`
package installed)

-   **Read files using multiple engine from multiple path and multiple
    file extension.**

Using [`purrr::pmap`](https://purrr.tidyverse.org/reference/map2.html)
in combination with `rd()`

Now, you can customize to read any files from any directory with any
file reading function you want !!

``` r
params <- list(
  .f = c(~read_csv(.x, col_names = F), readxl::read_excel),
  path = c("path/to/dir1", "path/to/dir2"), # `path` can be individual file as well 
  pattern = c("\\.csv$", "\\.xlsx")
  )

purrr::pmap(params, rd)
```

## Note

The `rd()` function was created as a wrapper for possibly any reading
functions using functional programming to read multiple files.

**The main goal is to provide a simple and easy way read multiple files
from a directory.**

However, if you want to further customize file reading beyond this
function can do, have a look at

[`fs`](https://fs.r-lib.org) package for more advance files system
operation.

<br>

Argument `.f` of `rd()` actually can be **any function** that take file
path as input (similar to
[`fs::dir_map`](https://fs.r-lib.org/reference/dir_ls.html) and
[`fs::dir_walk`](https://fs.r-lib.org/reference/dir_ls.html) )

However, you should use `rd()` to read files because it was optimized in
this way.
