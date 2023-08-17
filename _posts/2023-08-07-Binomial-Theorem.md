# The Binomial Theorem - raising binomials to powers

$$(a+b)^2 = (a^2 + 2ab + b^2)$$

$$(a+b)^3 = (a^3 + 3a^2b + 3ab^2 + b^3)$$

$$(a+b)^4 = (a^4 + 4a^3b + 6a^2b^2 + 4ab^3 + b^4)$$

So there's a pattern.  

## Variable Portion of each term

If $$n$$ is the power we're raising the binomial to, then for each subsequent term in the resulting polynomial, the exponent of $$a$$ starts at $$n$$ and goes down to $$0$$, while the exponent of $$b$$ starts at $$0$$ and climbs to $$n$$.

Also, for any $$n$$, you end up with a polynomial with $$n + 1$$ terms.

So, zero-indexing the terms where $$n$$ is the power, and $$k$$ is the zero-based index of the term, each term's variable portion is:
$$a^{n-k}b^{k}$$

## Coefficients

If we lay out all of the polynomials obtained by taking $$a+b$$ from power $$0$$ and up:
$$1$$
$$a+b$$
$$a^2 + 2ab + b^2$$
$$a^3 + 3a^2b + 3ab^2 + b^3$$
$$a^4 + 4a^3b + 6a^2b^2 + 4ab^3 + b^4$$

...and then extract just the coefficients, we get
$$1$$
$$1 1$$
$$1 2 1$$
$$1 3 3 1$$
$$1 4 6 4 1$$

Which is pascal's triangle.  And, taking the row of that triangle whose zero-based index is equal to the power we're after, we get the correct coefficients.

## Why?

When we're calculating a binomial to a power, it expands to a multiplication of binomials:
$$(a+b)^3 = (a+b)(a+b)(a+b)$$

When you multiply binomials, you're saying "every term in every binomial needs to be multiplied by every term in every other binomial". To do this manually, you'd go through a systematic approach:
$$
(a+b)(a+b)(a+b) 
= (a \times a \times a) + (a \times a \times b ) + (a \times b \times a) + (a \times b \times b) + ... + (b \times b \times b)
$$

So, all possible combinations of terms whose powers add up to $$n$$ will exist in our answer:
- $$a^3$$
- $$a^2b$$
- $$ab^2$$
- $$b^3$$

Some of them will occur more than once.  We'll only get one $$a^3$$, but we'll get three $$ab^2$$.

Each term's coefficient is determined by how many times that combination arises in the full multiplying-out. for a value of $$(a + b)^5$$, the factor $$(a+b)$$ is multiplied $$5$$ times:
$$(a+b)(a+b)(a+b)(a+b)(a+b)$$

Pascal's triangle gives the correct values for the coefficient values for the given power and term index; however, so does $${n \choose k}$$.

The function $${n \choose k}$$ ("$$n$$ choose $$k$$") is a function that returns the number of possible distinct sets of $$k$$ elements from a set of $$n$$.  So, if you have a set of 3 elements, $${3 \choose 1}$$ will give you 3: for any set of 3 elements, there are three subsets that contain one element.

Let's randomly choose some term that will be in the result, say $$a^2b^3$$.  What's the coefficient? We know that, when multiplying out, we would have chosen the $$a$$ from two of the $$(a+b)$$ factors, and the $$b$$ from the remaining three. So, given a subset of $$5$$ factors, how many distinct subsets are there with three members? It's an $${n \choose k}$$ problem.

Also, note that $$5 \choose 3$$ returns the same as $$5 \choose 2$$ ($$10$$), hence the palindromic shape of a line of Pascal's triangle.

### Proof for $${n \choose k}$$

This proof relies on thinking about the total possible set of orderings of any given set.

To calculate $${n \choose k}$$, imagine the total possible set of orderings of the set $$n$$ as a space. There will be $$n!$$ orderings in this space.

Now, imagine that we figure out by some means the complete set of subsets of $$k$$ elements from $$n$$. If we have a set of (1, 2, 3), the total set of $${3 \choose 2}$$ subsets is (1, 2), (1, 3) and (2,3). We then break the total ordering space up into groups: one group for each possible subset. That group of orderings will contain all of the orderings of $$n$$ where the first $$k$$ numbers are that set.

In our example:
* Group 1:
    * (1,2)(3)
    * (2,1)(3)
* Group 2:
    * (1,3)(2)
    * (3,1)(2)
* Group 3:
    * (2,3)(1)
    * (3,2)(1)
    
We'll have $${n \choose k}$$ groups, because that's how many subsets we have. So how many orderings are going to be in each group?

Each $$k$$ element prefix will have $$k!$$ possible orderings. Each $$n-k$$ element suffix will have $$(n-k)!$$ possible orderings; so, each group will have $$k!(n-k)!$$ possible orderings.

As the complete set is divide up into $${n \choose k}$$ groups, we now know that:

$$n! = {n \choose k}k!(n-k)!$$

Solve for $${n \choose k}$$, and you get

$$ {n \choose k} = {n! \over {k!(n-k)!}} $$

...which is the $${n \choose k}$$ formula.


## In an equation
So, any expression of $$(a+b)^n$$ will give rise to a set of terms: each will be some combination of the variables where the powers add up to n, and the coefficient will be equal to $$n \choose k$$, where $$n$$ is the power being raised to and $$k$$ is the power of any one of the variables:
$${n \choose k}(a^{n-k}b^k)$$

The final answer will be the sum of all such terms, with $$k$$ starting at $$0$$ and ending at $$n$$:
$$(a+b)^k = \sum_{k=0}^{n}{ {n \choose k}(a^{n-k}b^k) }$$
# The Binomial Theorem - raising binomials to powers

$$(a+b)^2 = (a^2 + 2ab + b^2)$$

$$(a+b)^3 = (a^3 + 3a^2b + 3ab^2 + b^3)$$

$$(a+b)^4 = (a^4 + 4a^3b + 6a^2b^2 + 4ab^3 + b^4)$$

So there's a pattern.  

## Variable Portion of each term

If $$n$$ is the power we're raising the binomial to, then for each subsequent term in the resulting polynomial, the exponent of $$a$$ starts at $$n$$ and goes down to $$0$$, while the exponent of $$b$$ starts at $$0$$ and climbs to $$n$$.

Also, for any $$n$$, you end up with a polynomial with $$n + 1$$ terms.

So, zero-indexing the terms where $$n$$ is the power, and $$k$$ is the zero-based index of the term, each term's variable portion is:
$$a^{n-k}b^{k}$$

## Coefficients

If we lay out all of the polynomials obtained by taking $$a+b$$ from power $$0$$ and up:
$$1$$
$$a+b$$
$$a^2 + 2ab + b^2$$
$$a^3 + 3a^2b + 3ab^2 + b^3$$
$$a^4 + 4a^3b + 6a^2b^2 + 4ab^3 + b^4$$

...and then extract just the coefficients, we get
$$1$$
$$1 1$$
$$1 2 1$$
$$1 3 3 1$$
$$1 4 6 4 1$$

Which is pascal's triangle.  And, taking the row of that triangle whose zero-based index is equal to the power we're after, we get the correct coefficients.

## Why?

When we're calculating a binomial to a power, it expands to a multiplication of binomials:
$$(a+b)^3 = (a+b)(a+b)(a+b)$$

When you multiply binomials, you're saying "every term in every binomial needs to be multiplied by every term in every other binomial". To do this manually, you'd go through a systematic approach:
$$
(a+b)(a+b)(a+b) 
= (a \times a \times a) + (a \times a \times b ) + (a \times b \times a) + (a \times b \times b) + ... + (b \times b \times b)
$$

So, all possible combinations of terms whose powers add up to $$n$$ will exist in our answer:
- $$a^3$$
- $$a^2b$$
- $$ab^2$$
- $$b^3$$

Some of them will occur more than once.  We'll only get one $$a^3$$, but we'll get three $$ab^2$$.

Each term's coefficient is determined by how many times that combination arises in the full multiplying-out. for a value of $$(a + b)^5$$, the factor $$(a+b)$$ is multiplied $$5$$ times:
$$(a+b)(a+b)(a+b)(a+b)(a+b)$$

Pascal's triangle gives the correct values for the coefficient values for the given power and term index; however, so does $${n \choose k}$$.

The function $${n \choose k}$$ ("$$n$$ choose $$k$$") is a function that returns the number of possible distinct sets of $$k$$ elements from a set of $$n$$.  So, if you have a set of 3 elements, $${3 \choose 1}$$ will give you 3: for any set of 3 elements, there are three subsets that contain one element.

Let's randomly choose some term that will be in the result, say $$a^2b^3$$.  What's the coefficient? We know that, when multiplying out, we would have chosen the $$a$$ from two of the $$(a+b)$$ factors, and the $$b$$ from the remaining three. So, given a subset of $$5$$ factors, how many distinct subsets are there with three members? It's an $${n \choose k}$$ problem.

Also, note that $$5 \choose 3$$ returns the same as $$5 \choose 2$$ ($$10$$), hence the palindromic shape of a line of Pascal's triangle.

### Proof for $${n \choose k}$$

This proof relies on thinking about the total possible set of orderings of any given set.

To calculate $${n \choose k}$$, imagine the total possible set of orderings of the set $$n$$ as a space. There will be $$n!$$ orderings in this space.

Now, imagine that we figure out by some means the complete set of subsets of $$k$$ elements from $$n$$. If we have a set of (1, 2, 3), the total set of $${3 \choose 2}$$ subsets is (1, 2), (1, 3) and (2,3). We then break the total ordering space up into groups: one group for each possible subset. That group of orderings will contain all of the orderings of $$n$$ where the first $$k$$ numbers are that set.

In our example:
* Group 1:
    * (1,2)(3)
    * (2,1)(3)
* Group 2:
    * (1,3)(2)
    * (3,1)(2)
* Group 3:
    * (2,3)(1)
    * (3,2)(1)
    
We'll have $${n \choose k}$$ groups, because that's how many subsets we have. So how many orderings are going to be in each group?

Each $$k$$ element prefix will have $$k!$$ possible orderings. Each $$n-k$$ element suffix will have $$(n-k)!$$ possible orderings; so, each group will have $$k!(n-k)!$$ possible orderings.

As the complete set is divide up into $${n \choose k}$$ groups, we now know that:

$$n! = {n \choose k}k!(n-k)!$$

Solve for $${n \choose k}$$, and you get

$$ {n \choose k} = {n! \over {k!(n-k)!}} $$

...which is the $${n \choose k}$$ formula.


## In an equation
So, any expression of $$(a+b)^n$$ will give rise to a set of terms: each will be some combination of the variables where the powers add up to n, and the coefficient will be equal to $$n \choose k$$, where $$n$$ is the power being raised to and $$k$$ is the power of any one of the variables:
$${n \choose k}(a^{n-k}b^k)$$

The final answer will be the sum of all such terms, with $$k$$ starting at $$0$$ and ending at $$n$$:
$$(a+b)^k = \sum_{k=0}^{n}{ {n \choose k}(a^{n-k}b^k)}$$
