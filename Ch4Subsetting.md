4.1 Introduction

-there are six ways to subset atomic vectors -there are three subsetting
operators \[\[, \[, $ -subsetting operators interact differently with
different vector types -subsetting can be combined with assignment
-str() shows you all the pieces of any object (its structure) and
subsetting allows you to pull out the pieces you’re interested in

Quiz

1.  What is the result of subsetting a vector with positive integers,
    negative integers, a logical vector, or a character vector?

2.  What’s the difference between \[\[, \[, $ when applied to a list?

3.  When should you use drop = FALSE?

4.  If x is a matrix, what does x\[\] &lt;- 0 do? How is it different
    from x &lt;- 0?

5.  How can you use a named vector to relabel categorical variables?

4.2 Selecting Multiple Elements

\[: used to select any number of elements from a vector

4.2.1 Atomic Vectors

**note that the number after the decimal point represents the original
position in the vector**

    x <- c(2.1, 4.2, 3.3, 5.4)
