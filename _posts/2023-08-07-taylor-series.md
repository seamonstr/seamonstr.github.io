# Taylor Series

[This video](https://www.youtube.com/watch?v=3d6DsjIBzJ4)

Given a quadratic equation $$f(x)=ax^2 + bx +c$$, the impact of the constants is:
* a = the "steepness" of the parabola
* b = has a disproportionate effect when $$x$$ is small. Biggest impact is at the y-intercept (ie. $$x \to 0$$), so it determines the derivitave of $$x$$ at the intercept.
* c = the intercept.

## Approximating a function with a polynomial

* You can set the intercept by setting $$c$$
* Set the highest y of the parabola to a given value by setting its derivative to be 0 at that point; if you don't know the intercept, this gives you a function that relates $$a$$ and $$b$$. Plugging values for that in lets you solve for $$c$$.
* Set the curve by setting the second derivative at the turnaround to be the same as that of the function you're approximating

The zen of powers and coefficients in an equation is that the higher powers have a greater effect with larger values of x.  So, you can use  the lower powers to give you the curve early on, then the higher powers later but with very small co-efficients to minimise their impact in the early stages.

## At $$x=0$$ only one term impacts any derivative
Adding higher order terms to a polynomial has no effect on the derivatives of that polynomial at $$x = 0$$. 
Eg. for $$f(x) = ax^4 + bx^3 + cx^2 + dx + e$$, if you take the second derivative then the power rule will take $$dx$$ and $$e$$ to zero, the $$cx$$ term will become a constant (because it'll contain $$x^0$$), and the higher order terms will evaluate to zero because they still have an $$x$$ in them.

So, at $$x = 0$$, only one term in a polynomial controls a given derivative.

To achieve the same effect at values of $$x$$ that are not zero, you have to offset the value of $$x$$ by the value you're optimising for. In this case, it's $$\pi$$:

$$f(x) = a + b(x - \pi) + c(x^2 - \pi) + ...$$

## Taylor polynomials

A Taylor series is an infinite series expressed in terms of a function's derivatives at a particular point, starting at with the function itself and then summing the higher order derivatives at that point. In the following, $$a$$ is the point the series is optimised for:

$$g(x) = f(a) + {f'(a).x \over 1!} + {f''(a).x \over 2!} + {f'''(a).x \over 3!} + ...$$

So, the derivatives of cosine and their values at $$x=0$$ are:

$$\cos(0) = 1$$

$$-\sin(0) = 0$$

$$-\cos(0) = -1$$

$$\sin(0) = 0$$

$$\cos(0) = 1$$

$$...$$

...and putting this into a polynomial that approximates $$cos$$ at x = 0 gives us the Taylor polynomial:

$$f(x) = 1 + 0{ x^1 \over 1! } + -1{ x^2 \over 2! } + 0{ x^3 \over 3! } + 1{ x^4 \over 4! } + ... $$

Each derivative of this polynomial, at $$x = 0$$, will only return a value from the single term that corresponds to that derivative due to the cancelling out effect above. 

Put generally, if you want to approximate a given function $$g$$ at some value of x, you do

$$f(x) = g(x) + {dg \over dx}(0){x^1 \over 1!} + {d^2g \over dx^2}(0){x^2 \over 2!} + {d^3g \over dx^3}(0){x^3 \over 3!} + ... $$

ie. compute all of the derivatives of the function you want to approximate. Each nth term of the approximation function becomes the nth derivative of the original function, evaluated at $$x=0$$ (or some other value, if that's where you're optimising), divided by $$n!$$.

The same function, including the possibility of optimising at some value of $$x$$ other than 0 ($$a$$):
$$f(x) = g(x) + {dg \over dx}(a){(x - a)^1 \over 1!} + {d^2g \over dx^2}(a){(x - a)^2 \over 2!} + {d^3g \over dx^3}(a){(x - a)^3 \over 3!} + ... $$

All of the terms in a taylor polynomial, to $$\infty$$, are known as a Taylor series. For some functions, such as $$f(x) = a^x$$, the Taylor series will actually completely converge on the function for all possible values of x. We say that the function equals its Taylor series in this case.

For others, it only converges within a small distance of the optimisation point; this range is called the radiance of convergence for the Taylor series.