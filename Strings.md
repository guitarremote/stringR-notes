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

### Defining strings

Defning string in R is a bit tricky and there are some conventions to handle different kind of scenarios.

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

Numbers can be converted to strings using `format` and `formatC`. The numbers can be converted to two types; fixed and scientific.

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

### Stringr

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
-   `str_subset()`- returns only the strings from the vector that contains the pattern
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

**Replacing strings**

`str_replace` and `str_replace_all`

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

### Regular expressions

Regular expressions are a language for desribing patterns. Take the pattern below and its meaning for instance <br/>

`^.[\d]+`- "the start of the string, followed by any single character, followed by one or more digits" <br/>

The same regex pattern can be obtained using the `rebus` library. The syntax is a bit more verbose but more understandable. Below is the syntax for the same regex using `rebus` : <br/>

``` r
START %R%
  ANY_CHAR %R%
  one_or_more(DGT)
```

    ## <regex> ^.[\d]+

`rebus` provides `START` and `END` shortcuts to specify regular expressions that match the start and end of the string. These are also known as *anchors*. <br/>

The special operator provided by rebus, `%R%` allows you to compose complicated regular expressions from simple pieces. When you are reading rebus code, think of `%R%` as "then".<br/>

`START %R% "c"` - this regex should be interpreted as "the start of a string then a c" <br/>

`str_view()` from `stringr` is really helpful for testing patterns. It will show an html output with the matches highlighted.

``` r
pattern <- "z" %R% ANY_CHAR

# Find names that have the pattern
names_with_z <- str_subset(boy_names_2014,pattern=pattern)
length(names_with_z)
```

    ## [1] 441

``` r
# Find part of name that matches pattern
part_with_z <- str_extract(boy_names_2014,pattern=pattern)
table(part_with_z)
```

    ## part_with_z
    ##  za  zc  zd  ze  zg  zh  zi  zj  zk  zl  zm  zn  zo  zp  zr  zu  zv  zw 
    ## 133   2   4 108   1   4 104   1   1   5   7   1  22   2  12   5   1   2 
    ##  zy  zz 
    ##  10  16

``` r
# Did any names have the pattern more than once?
count_of_z <- str_count(boy_names_2014,pattern=pattern)
table(count_of_z)
```

    ## count_of_z
    ##     0     1     2 
    ## 13585   439     2

``` r
# Which babies got these names?
with_z <- str_detect(boy_names_2014,pattern=pattern)

# What fraction of babies got these names?
mean(with_z)
```

    ## [1] 0.03144161

### More on regex

| Pattern                           | Regular Expression      | rebus            |
|-----------------------------------|-------------------------|------------------|
| Start of a string                 | ^                       | START            |
| End of a string                   | $                       | END              |
| Any single character              | .                       | ANY\_CHAR        |
| Literal dot, carat or dollar sign | `"\\."`,`"\\^"`,`"\\$"` | DOT,CARAT,DOLLAR |

**Alternation** - This is more like a logical OR, regex `"(dog|cat)"` will capture "dog" or "cat". The `rebus` equivalent of these patterns is the `or` function.<br/>

**Character classes** - Similar to `ANY_CHAR` but with a restricton on what characters are allowed. Regex expressions patterns look like `[Aa]` or `[^Aa]`. The carat inside a class means negated. The `rebus` equivalent functions for these patterns are `char_class()` and `negated_char_class()`. <br/>

**Repititions** - In regex the metacharacters to handle repititions are:

-   `?`- optional
-   `*`-zero or more
-   `+`-one or more
-   `{n}{m}`-repeated between n and m times

The `rebus` equivalent functions for these patterns are `optional()`,`zero_or_more`,`one_or_more()`,`repeated()` respectively.<br/>

``` r
#match names that start with Cath or Kath
ckath <- or("C","K")%R%"ath"
cath_names <- str_subset(girl_names_2014,pattern=ckath)
print(head(cath_names))
```

    ## [1] "Katherine" "Catherine" "Kathryn"   "Kathleen"  "Kathy"     "Katharine"

``` r
#names that are only vowels
vowels<-char_class("aeiouAEIOU")
vowel_names <- str_subset(boy_names_2014,pattern = exactly(one_or_more(vowels)))
print(head(vowel_names))
```

    ## [1] "Io"

``` r
#negated_char_class() for names without any vowels
not_vowels<-negated_char_class("aeiouAEIOU")
no_vowel_names <- str_subset(boy_names_2014,pattern = exactly(one_or_more(not_vowels)))
print(head(no_vowel_names))
```

    ## [1] "Ty"    "Rhys"  "Flynn" "Sky"   "Fynn"  "Kyng"

### Shortcuts

Suppose we want to match any digit, rather than writing all the digits from 0 to 9, there are shortcuts available in regex to take care of that. The shortcut for digits is \[0-9\]. Below are a few more examples:

-   `[0-9]` or `\d`- A digit - `rebus` equivalent - `char_class("0-9")` or `DGT`
-   `[a-z]`- A lower case letter - `rebus` equivalent - `char_class("a-z")`
-   `[A-Z]` - An upper case letter - `rebus` equivalent - `char_class("A-Z")`
-   `\w` - A word character - `rebus` equivalent - `char_class("a-zA-Z0-9")` or `WRD`
-   `\s` - A whitespace character - `rebus` equivalent - `SPC`

### Advanced regex tricks

#### Capturing parts of a pattern

In rebus, to denote a part of a regular expression you want to capture, you surround it with the function `capture()`. In the example below, we are capturing the part before a `@` in an email. The part of the string that matches hasn't changed, that is the reason why the output of `str_extract` is the same below for both `email_pattern` and `email_capture_pattern`. But if we pull out the match using `str_match`,we see something interesting.

As you can see for `email_capture_pattern`, `str_match` returns a matrix. The first column is the match of the total pattern, which is same as `email_pattern`. The second column is the match of the captured part. This comes in very handy when we create a complicated pattern but still want to access different parts of it.

``` r
test_email <- "My email is john_cena@wwe.com. You can't see me"
#pattern to detect an email
email_pattern <- one_or_more(WRD) %R% 
  "@" %R% one_or_more(WRD) %R% 
  DOT %R% one_or_more(WRD)

#same pattern with a capture
email_capture_pattern <- capture(one_or_more(WRD)) %R% 
  "@" %R% one_or_more(WRD) %R% 
  DOT %R% one_or_more(WRD)

email_pattern
```

    ## <regex> [\w]+@[\w]+\.[\w]+

``` r
email_capture_pattern
```

    ## <regex> ([\w]+)@[\w]+\.[\w]+

``` r
str_extract(test_email,pattern = email_pattern)
```

    ## [1] "john_cena@wwe.com"

``` r
str_extract(test_email,pattern = email_capture_pattern)
```

    ## [1] "john_cena@wwe.com"

``` r
str_match(test_email,pattern = email_pattern)
```

    ##      [,1]               
    ## [1,] "john_cena@wwe.com"

``` r
str_match(test_email,pattern = email_capture_pattern)
```

    ##      [,1]                [,2]       
    ## [1,] "john_cena@wwe.com" "john_cena"
