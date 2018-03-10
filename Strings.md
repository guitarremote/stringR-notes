String manipulation in R
================
Atreya
4 March 2018

``` r
library(stringr)
library(babynames)
library(dplyr)
library(rebus)
```

Some guidelines to entering strings

``` r
l1 <- "I said Hi!" # No quotes in the strng, use double quotes
l2 <- 'I said "Hi"'# Double quotes in the string, use single quotes
l3 <- "I'd said \"Hi!\"" #Doubles and single quotes in the string, use double quotes and escape sequences
lines <- c(l1,l2,l3)
writeLines(lines,sep="\n")
```

    ## I said Hi!
    ## I said "Hi"
    ## I'd said "Hi!"

Turning numbers to string using `format` and `formatC`. Two number types are fixed and scientific

``` r
nums <- c(19800000000000000000000000000,0.000000000000000000000023)
format(nums,scientific = TRUE)
```

    ## [1] "1.98e+28" "2.30e-23"

``` r
format(nums,scientific = FALSE)
```

    ## [1] "19800000000000000977776148480.000000000000000000000000"
    ## [2] "                            0.000000000000000000000023"

``` r
formatC(nums,format = "f")#fixed
```

    ## [1] "19800000000000000977776148480.0000"
    ## [2] "0.0000"

``` r
formatC(nums,format = "e")#scientific
```

    ## [1] "1.9800e+28" "2.3000e-23"

``` r
formatC(nums,format = "g")#scientific only if it saves space
```

    ## [1] "1.98e+28" "2.3e-23"

The format function works in a tricky way. When the representation is scientific, the digits argument is the number of digits before the exponent. When the representation is fixed, digits controls the significant digits used for the smallest (in magnitude) number. Each other number will be formatted to match the number of decimal places in the smallest number. This means the number of decimal places you get in your output depends on **all** the values you are formatting!

``` r
percent_change  <- c(4, -1.91, 3.00, -5.002)
income <-  c(72.19, 1030.18, 10291.93, 1189192.18)
p_values <- c(0.12, 0.98, 0.0000191, 0.00000000002)
format(percent_change,digits = 2)
```

    ## [1] " 4.0" "-1.9" " 3.0" "-5.0"

``` r
format(income,digits = 2)
```

    ## [1] "     72" "   1030" "  10292" "1189192"

``` r
format(p_values,digits = 2)
```

    ## [1] "1.2e-01" "9.8e-01" "1.9e-05" "2.0e-11"

`paste` can be used to join strings

``` r
paste(c("Hey","How are"),c("you!","you?"))
```

    ## [1] "Hey you!"     "How are you?"

``` r
paste(c("Hey","How are"),c("you!"),sep="*",collapse = ", ")
```

    ## [1] "Hey*you!, How are*you!"

**Stringr**

-   Built on `stringi`, easy to learn
-   All functions start with `str_`, concise and consistent
-   All stringr functions use a vector of strings as a first argument

``` r
#difference between str_c and paste
s1<- c("cards",NA,NA)
s_paste_1 <- paste(c("", "", "and "), s1, sep = "")
s_paste_2 <- paste(c("", "", "and "), s1, collapsse = ", ")
s_str_1 <- str_c(c("", "", "and "), s1, sep = "")
s_str_2 <- str_c(c("", "", "and "), s1, collapse = "")
s_paste_1
```

    ## [1] "cards"  "NA"     "and NA"

``` r
s_paste_2
```

    ## [1] " cards , "  " NA , "     "and  NA , "

``` r
s_str_1 #str_c propogates NA unline paste
```

    ## [1] "cards" NA      NA

``` r
s_str_2
```

    ## [1] NA

Let's work on the babynames dataset

``` r
boy_names_2014 <- filter(babynames,year==2014 & sex=="M")$name
girl_names_2014 <- filter(babynames,year==2014 & sex=="F")$name
str_length(head(boy_names_2014))#length
```

    ## [1] 4 4 5 5 7 5

``` r
str_sub(head(boy_names_2014),1,3)#extracting subtrings
```

    ## [1] "Noa" "Lia" "Mas" "Jac" "Wil" "Eth"

``` r
str_sub(head(girl_names_2014),-3,-1)#extracting subtrings
```

    ## [1] "mma" "via" "hia" "lla" "Ava" "Mia"

**Detecting patterns**

-   `str_detect()`- returns a logical vector indicating whether the pattern exists or not
-   `str_subset()`- returns only the strng from the vector that contains the pattern
-   `str_count()`- returns the number of times a pattern occured in each element of the vector

``` r
contains_ara <- str_detect(boy_names_2014,pattern = fixed("ara")) #case is important, by default ignore_case is FALSE for fixed
head(boy_names_2014[contains_ara])
```

    ## [1] "Aarav"   "Pharaoh" "Karas"   "Ciaran"  "Karam"   "Taran"

``` r
str_subset(str_subset(girl_names_2014,pattern=fixed("U")),pattern=fixed("z"))
```

    ## [1] "Umaiza"

``` r
girl_names_2014[str_count(girl_names_2014,pattern =fixed("a",ignore_case = TRUE))>4]
```

    ## [1] "Aaradhana"

**Splitting strings**

`str_split()`- `simplify`=TRUE will return a matrix, else a list would be returned, `n`- parameter allows the number of splits

``` r
both_names <- c("Box, George", "Cox, David")
both_names_split <- str_split(both_names,pattern=fixed(", "),simplify=TRUE)
first_names <- both_names_split[,2]
last_names <- both_names_split[,1]
```

\*\* Replacing strings \*\* `str_replace` and `str_replace_all`

``` r
phone_numbers <- c("510-555-0123", "541-555-0167")
str_replace(phone_numbers,pattern=fixed("-")," ")
```

    ## [1] "510 555-0123" "541 555-0167"

``` r
# Use str_replace_all() to replace "-" with " "
str_replace_all(phone_numbers,pattern=fixed("-")," ")
```

    ## [1] "510 555 0123" "541 555 0167"

##### Regular expressions
