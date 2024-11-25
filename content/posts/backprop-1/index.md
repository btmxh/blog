+++
date = '2024-11-23T23:39:12+07:00'
title = 'backprop-1.md'
+++

## My earlier attempts with Machine Learning

When I started programming, I saw all sort of videos about training neural networks
to play games. Y'know, those "AI learns to play *name of game*" videos that use
some black-box optimization techniques like Genetic Algorithm (GA) to optimize the
network and gradually gets better at playing that game. The most iconic video was
probably SethBling's [MarI/O - Machine Learning for Video Games](https://www.youtube.com/watch?v=qv6UVOQ0F44)
(which, not gonna lie, was much simpler than what I recalled).
This technique originates from an old paper,
[Evolving Neural Networks through Augmenting Topologies](https://direct.mit.edu/evco/article-abstract/10/2/99/1123/Evolving-Neural-Networks-through-Augmenting),
which I have just seen right now after checking out the SethBling video's description.
The main idea is to prepare some neural network topology, and the standard procedure of GA is used to
furthermore optimize this model via tweaking its parameters.

GA is inefficient, to say the least. Neural networks have a quite simple structure, it's
just the scale that is the problematic part, so using a black-box technique on training
seem like a wasteful decision. Back to the past, I was in high (or middle? I don't really
remember it well) school programming games with OpenGL and just came across
[this legendary video series](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi)
of 3Blue1Brown, which actually explained in detail what a neural network is and how to
train it using a more up-to-date method, Backpropagation and Gradient Descent.
I did not quite understand the math back then, but once I nailed down high school calculus,
it all made sense to me.

Before entering university, I decided to put my knowledge of Calculus to good use and tried to
program a simple [MNIST](https://en.wikipedia.org/wiki/MNIST_database) digit classifier program,
as a warm-up (back then, I thought university classes would teach more impressive stuff than I
thought, and being able to program a simple ML demo should be a minimum entry point). Then, I
arrived at a roadblock. I couldn't figure out the math for the matrix operations. I had a good
sense for Linear Algebra thanks for 3Blue1Brown's
[series](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab), but I just
could not derive the math. Back then, all I know about matrix multiplication was just
"oh matrix multiplication is just linear map composition", and I had no experience of actually
doing the computation. Unlike a sensible undergrad who
add indices to everything an reduce the problem to a scalar one, I took the harder approach
and attempt to study matrix calculus. There, I found an amazing lecture notes PDF by Joseph
Breen on Advanced Multivariable Differential Calculus. Back then, you can search the file on
Google and find a link, but it seems like they deleted it.

After studying everything in that PDF, I became obsessed with the concept of the Fréchet
derivative, which is a generalization of a derivative in normed vector spaces. It basically
changed my mind on what's really a derivative. The rate-of-change intuition makes sense
in scalar space \(\mathbb{R}\), but it broke down completely in higher-dimensional spaces,
where vector division no longer makes sense.

Before continuing, I advise you to study up on basic Linear Algebra and Calculus (multivariate,
preferably), as well as getting a good sense of what Backpropagation actually do under the
hood. You don't have the know the algebraic techniques of finding limits and solve linear
equations, just a good sense of intuition is fine.

## So, what's the proper way to represent the derivative?

Given a function \(f: V \to W\) that maps from a normed vector space \(V\) to another normed
vector space \(W\). Wait, what is a normed vector space?

A vector space is simply a space of vector-like objects. Head to
[Wikipedia](https://en.wikipedia.org/wiki/Vector_space) for the rigorous definition,
but here I will just explain it intuitvely. Vector-like objects are object you can:
- add (or subtract) with a vector-like object of the same "type" (in the same space), or
- multiply with a scalar, i.e. a real number,

in a way that preserves some sensible conditions (associativity, commutativity, distributivity,
etc.).

Then,
- \(\mathbb{R}\), the set of real numbers is a vector-space.
- \(\mathbb{R^{n}}\), the set of \(n\)-dimensional vectors is also a vector-space.
- \(\mathbb{R^{m \times n}}\), the set of \(m \times n\)-dimensional matrices is yet another
  vector-space.

Now, if we add some division operation to our definition above, everything completely broke
down. There is just no conventional way to divide vectors or even matrices.

Next, we define a norm as a measure of distance from some origin (the zero vector),
denoted as \(\|\cdot\|\), such that:

- Norm of a vector must be always non-negative: \( \|\mathbf{v}\| \ge 0 \). Moreover,
  equality only holds if \( \mathbf{v} \) is 0-distance away from the origin, i.e.
  \( \mathbf{v} \) is the origin itself.
- The triangle inequality holds: \( \|\mathbf{v} + \mathbf{w}\| \le \|\mathbf{v}\| + \|\mathbf{w}\| \).
- Scaling a vector also scale its norm: \( \|k \cdot \mathbf{v}\| = |k| \cdot \|\mathbf{v}\| \).

Then, a normed vector space is just a vector space with a defined norm. With that out of the way,
we can go back to the definition.

Given a function \(f: V \to W\) that maps from a normed vector space \(V\) to another normed
vector space \(W\) and a point \( \mathbf{v} \in V \). Then, if a linear map \( L: V \to W \)
is called the derivative of \( f \) at \( \mathbf{v} \) iff this limit exists and holds:

\[ \lim_{d \mathbf{v} \to 0} \frac{\|f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) - L(d \mathbf{v})\|}{\|d \mathbf{v}\|} = 0. \]

Here, \( d \mathbf{v} \) is just another variable, which might look confusing, but this notation
will be useful later on.

When I say \( L \) is a linear map, it means that you can do stuff like
\( L(\mathbf{x} + \mathbf{y}) = L(\mathbf{x}) + L(\mathbf{y}) \) and
\( L(k \mathbf{x}) = k L(\mathbf{x}) \).
So \( L(d \mathbf{v}) = f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) \),
even though it satisfies the limit, is not a valid derivative (in most cases).

Now, the derivative of a function is a function. Moreover, it's the same kind of function
as the original one! Why is it this way?

The limit basically says that the numerator approaches 0 faster than \( d \mathbf{v} \),
which means \( f(\mathbf{v}) + L(d \mathbf{v}) \) is very good at approximating
\( f(\mathbf{v} + d \mathbf{v}) \) (for small \( d \mathbf{v} \)). In other words, when you
zoom in the graph of \( f \) at \( \mathbf{v} \), the graph will resemble that of \( L \)
(at anywhere, since \( L \) is linear).

If we denote \( d f = f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) \),
then \( d f \approx L(d \mathbf{v}) \). \( L \) is the **linear approximation** to
\( df \), which can be thought of as the change in \( f \) when one add a small amount
\( d \mathbf{v} \) to \( \mathbf{v} \).

So, we replace the rate of change by the linear approximation of change. It does make sense,
if we have the rate of change, we can construct the linear approximation, but the rate of
change is not defined in higher-dimensional spaces. The Fréchet derivative solved this problem.

## Some basic results of Fréchet derivative

When you look at my wordings, you might realize that I'm calling \( L \) **the** derivative,
not a derivative. This suggests that \( L \) is unique, in the sense that if \( K \) and \( L \)
are derivatives of \( f \) at \( \mathbf{v} \), then \( K(d \mathbf{v}) = L(d \mathbf{v}) \) for
all \( d \mathbf{v} \).

Let's prove it, shall we?

Using the triangle inequality, we have:

\[
\begin{align*}
0 \le \frac{\|K(d \mathbf{v}) - L(d \mathbf{v})\|}{\|d \mathbf{v}\|} &\le
\frac{\|f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) - K(d \mathbf{v})\|}{\|d \mathbf{v}\|}\\
&+\frac{\|f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) - L(d \mathbf{v})\|}{\|d \mathbf{v}\|}
\end{align*}
\]

By the [squeeze theorem](https://en.wikipedia.org/wiki/Squeeze_theorem), we must have:

\[
\lim_{d \mathbf{v} \to 0} \frac{\|K(d \mathbf{v}) - L(d \mathbf{v})\|}{\|d \mathbf{v}\|} = 0.
\]

Taking the partial limit as \( d \mathbf{v} = t \mathbf{u}, t \to 0^+, \mathbf{u} \neq 0 \),
we have:

\[
\begin{align*}
0 &= \lim_{d \mathbf{v} \to 0} \frac{\|K(d \mathbf{v}) - L(d \mathbf{v})\|}{\|d \mathbf{v}\|}\\
&= \lim_{t \to 0} \frac{K(t \mathbf{u}) - L(t \mathbf{u})}{\|t \mathbf{u}\|}\\
&= \lim_{t \to 0} \frac{t K(\mathbf{u}) - t L(\mathbf{u})}{t \|\mathbf{u}\|}\\
&= \frac{K(\mathbf{u}) - L(\mathbf{u})}{\|\mathbf{u}\|}
\end{align*}
\]

This must mean \( K(\mathbf{u}) = L(\mathbf{u}) \). And this holds for all \( \mathbf{u} \).
Therefore \( K = L \).

With uniqueness out of the way, we continue with linearity. If \( K \) and \( L \)
are derivatives of \( f \) and \( g \) at \( x \), respectively, then
\( \alpha K + \beta L \) is the derivative of \( \alpha f + \beta g \) at \( x \).
Once again, the proof utilize triangle inequality and the squeeze theorem:

\[
\begin{align*}
0 &\le \frac{\| (\alpha f + \beta g)(\mathbf{x} + d \mathbf{x}) - (\alpha f + \beta g)(\mathbf{x}) - (\alpha K + \beta L)(d \mathbf{x}) \|}{\|d \mathbf{x}\|}\\
&= \frac{\|
\alpha (f(\mathbf{x} + d \mathbf{x}) - f(\mathbf{x}) - K(d \mathbf{x}))+
\beta (g(\mathbf{x} + d \mathbf{x}) - g(\mathbf{x}) - L(d \mathbf{x}))
\|} {\|d \mathbf{x} \|}\\
&\le
|\alpha| \cdot \frac{\|f(\mathbf{x} + d \mathbf{x}) - f(\mathbf{x}) - K(d \mathbf{x})\|}{\|d \mathbf{x}\|} +
|\beta| \cdot \frac{\|g(\mathbf{x} + d \mathbf{x}) - g(\mathbf{x}) - L(d \mathbf{x})\|}{\|d \mathbf{x}\|}\\
& \to 0, \text{as } d \mathbf{x} \to 0
\end{align*}
\]

Another important result is the derivative of linear and constant maps.

- If \( f \) is linear, than its derivative at anywhere is itself.
  Remember our attept at letting \( L(d \mathbf{v}) = f(\mathbf{v} + d \mathbf{v}) - f(\mathbf{v}) \)
  above? It turns out that when \( f \) is linear, we can expand
  \( f(\mathbf{v} + d\mathbf{v}) = f(\mathbf{v}) + f(d\mathbf{v}) \), yielding
  \( L(d \mathbf{v}) = f(d\mathbf{v}) \).
- If \( f \) is constant, than its derivative is the zero map. This should be easy to show,
  both intuitively and rigorously.

Using linearity of derivatives, we can find the derivative of any affine map,
i.e. the sum of a constant map and a linear map.

This might seem trivial, but I was truly shocked once I realize operations like trace
(\( \operatorname{tr} \)) and transpose are (\( \cdot^T \)) are linear. It turns out that
those operations actually belong to the class of easiest operations to differentiate!

## The heart of Backpropagation: the Chain Rule

The Chain Rule can be stated as follows.

Given functions \( f: U \to V, g: V \to W \) with \( U, V \) and \( W \) being normed vector spaces.
For some \( \mathbf{x} \in U \), if \( K \) is the derivative of \( f \) at \( \mathbf{x} \),
\( L \) is the derivative of \( g \) at \( f(\mathbf{x}) \),
then the composition of \( L \) and \( K \),

\[ M(d \mathbf{x}) = L(K(d \mathbf{x})), \]

is in fact the derivative of \( h(\mathbf{x}) = g(f(\mathbf{x})) \) at \( \mathbf{x} \),
assuming that \( K \) and \( L \) is **bounded**.

\( L \) is a bounded linear map is some linear map such that \( \|L(\mathbf{x})\| \) is always
bounded by \( \alpha \| \mathbf{x} \|\), with \( \alpha \) being an arbitrary constant.
If the domain and codomain of a linear map is finite-dimensional, then it is bounded.
This result is pretty complicated, so I will just prove it for the norm that we are going to
use: the Euclidean norm. The same logic also applies for the Frobenius norm, which will be
discussed below.

If \( \mathbf{x} = \sum_{k=1}^n x_k \mathbf{e}_k \), then
\( \|L(\mathbf{x})\| \le \sum_{k=1}^n |x_k| \|L(\mathbf{e}_k)\| \le \|\mathbf{x}\| \sum_{k=1}^n \|L(\mathbf{e}_k)\| \).
Hence, it suffices to just let \( \alpha = \sum_{k=1}^n \|L(\mathbf{e}_k)\| \).

With boundedness out of the way, we start with the proof.
We need to prove:

\[ \lim_{d \mathbf{x} \to 0} \frac{\|h(\mathbf{x} + d\mathbf{x}) - h(\mathbf{x}) - M(d\mathbf{x})\|}{\|d\mathbf{x}\|} = 0. \]

Let's look at the numerator:

\[
\begin{align*}
&h(\mathbf{x} + d\mathbf{x}) - h(\mathbf{x}) - M(d\mathbf{x})\\
&= (g(f(\mathbf{x} + d\mathbf{x})) - g(f(\mathbf{x})) - L(f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x})) ) + L(f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) - K(d\mathbf{x}))\\
&= (g(f(\mathbf{x}) + df) - g(\mathbf{x}) - L(df)) + L(f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) - K(d\mathbf{x})),
\end{align*}
\]

where \( df = f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) \).

Using the triangle inequality and the squeeze theorem, the problem reduces to proving:

\[ \lim_{d\mathbf{x} \to 0} \frac{\|g(f(\mathbf{x}) + df) - g(\mathbf{x}) - L(df)\|}{\|d\mathbf{x}\|} = 0, \]

and,

\[ \lim_{d\mathbf{x} \to 0} \frac{\|L(f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) - K(d\mathbf{x})\|}{\|d\mathbf{x}\|} = 0. \]

The first limit can be split to:

\[ \lim_{d\mathbf{x} \to 0} \left( \frac{\|g(f(\mathbf{x}) + df) - g(\mathbf{x}) - L(df)\|}{\|df\|}\cdot \frac{\|df\|}{\|d\mathbf{x}\|} \right) = 0 \]

The first term converges since \( df \to 0 \) when \( d\mathbf{x} \to 0 \) (why?), and the second
term is bounded:

\[ 0 \le \frac{\|df\|}{\|d\mathbf x\|} \le  \frac{\|f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) - K(d\mathbf x)\|}{\|d\mathbf{x}\|} + \frac{\|K(d\mathbf x)\|}{\|d\mathbf x\|}. \]

The second limit can be split to:

\[ \lim_{d\mathbf{x} \to 0} \left(\frac{\|L(d f - K(d\mathbf{x}))\|}{\|d f - K(d\mathbf{x})\|}\cdot\frac{\|f(\mathbf{x} + d\mathbf{x}) - f(\mathbf{x}) - K(d\mathbf{x})\|}{\|d\mathbf{x}\|}\right) = 0. \]

The second term clearly converges, while the first is bounded since \( L \) is bounded and \( df -K(d\mathbf x) \to 0 \) as \( d\mathbf x \to 0 \).

Phew, we are finally done.

## Total derivative and partial derivatives.

When we work with derivatives, working with linear maps directly is clunky, so we will use
total derivative notation to get around that.

Instead of working with functions like \( f \) and \( L \), we will talk about **quantities**.
Quantities form some form of dependency tree, like so:

\[ w = y^2 z, y = 2x, z = x^2. \]

The "leaf node" of such tree can be changed freely, while other quantity must change based on its
dependency. The \( d \) operator is used to denote the change amount of a quantity.

If \( dx \) is how much \( x \) changes, then we can easily calculate \( dy, dz, dw \):

\[ dy = 2dx, dz = 2xdx, dw = 16x^3dx. \]

As you can see, this not only depends on \( dx \), but also the current value of \( x \) itself.
In general, change amount depends on the base change amounts and the current value of the
quantities.

Using this notation, it is very easy to apply the chain rule:

\( dw = d(y^2 z) = d(y^2) z + y^2 dz = 2y\, dy\, z + y^2\, dz = 4x\cdot 2dx\cdot x^2 + (2x)^2\cdot 2x = 16x^3 dx. \)

Here, I cheated and used the product rule. Well, that's the second topic of this part.

Given a function \( f: V \to W \) with derivative \( df \) at some \( x \in V \), and the problem
is finding \( df \). To know how a linear map behave, a straightforward way is to see how it
acts on some basis of its domain.

Let me rephrase it a little bit differently.
Consider a variable quantity \( \mathbf{x} \) and \( n \) quantities that depend on it:
\( \mathbf{y}_1, \mathbf{y}_2, \ldots , \mathbf{y}_n \). Finally, we combined everything
into some sort of "super quantity": \( \mathbf{z} = f(\mathbf{y}_1, \mathbf{y}_2, \ldots , \mathbf{y}_n) \).
Now, the problem is to find the derivative of \( \mathbf{z} \) with respect to \( \mathbf{x} \).

\( f \) is a function that maps \( n \) vectors to a singular vector, you can always group
all \( n \) vectors into a single "supervector" and define some form of norm on that
"supervector" space. Then, \( f \) is a mapping from a normed vector space to another,
and \( df \) is now a well-defined concept.

A way to do this is to fix some \( i \) and let \( d\mathbf{y}_k = 0 \) for all \( k \neq i \).
Then, \( f \) collapses into a simpler map \( g \) whose only variable is \( \mathbf{y}_i \).
Assuming that we can find the derivative \( dg \) of \( g \), which is oftenly written as
\( \frac{\partial f}{\partial \mathbf{y}_i} \) in standard Calculus, then what we just did is
finding how \( df \) act on certain basis with respect to \( d\mathbf{y}_i \). Doing the same
process for other values for \( i \) yields:

\[ df = \sum_{i = 1}^n \frac{\partial f}{\partial \mathbf{y}_i} (d\mathbf{y}_i). \]

Now, how do we find this partial derivative? It's simple: just treat everything other than
\( \mathbf{y}_i \) a constant. For example, consider the problem of differentiating \( x^2 \)
with respect to \( x \). We will write the problem as:

\[ y = z = x, w = yz. \]

Then, \( dy = dz = dx \) and \( dw = \frac{\partial (yz)}{\partial y} (dy) + \frac{\partial (yz)}{\partial z} (dz). \)
\( \frac{\partial(yz)}{\partial y} (dy) \) is simply \( dy \cdot z \) since \( yz \) is a linear
map with respect to \( y \). This also simplifies further to \( xdx \), and so does the other term.
Hence, \( d(x^2) = 2xdx \).

Note that we did not use any knowledge of the power rule or anything similar in finding the
derivative of \( x^2 \). However, we did make a vital assumption that the derivative itself
exists. For this example, this can be easily verified by plugging back \( 2xdx \) to the
original definition, but this is not a general way to handle these circumstances. ML don't
really care about the nitty-gritty details of the derivative, since they pretend stuff
like \( \operatorname{relu} \) are differentiable anywhere, but if we want to be rigorous,
we need to be vary of that. There is, however, an useful result you can use for proving
the existence of derivative:

**Theorem**: If all partial derivatives of a function exists and is continuous in a neighborhood
of a point \( \mathbf{x} \), then that function has a derivative at \( \mathbf{x} \).

## Derivative of the squared Frobenius norm

Now, we will utilize everything we proved above.

The squared Frobenius norm of a \( m \times n \) matrix is defined as:

\[ \|M\|^2 = \sum_{k = 1}^m \sum_{l = 1}^n M_{kl}^2 \]

Doing it component-wise would be no fun, so let's write \( \|M\|^2 \) as:

\[ \|M\|^2 = \operatorname{tr}(M^T M). \]

Let use the \( d \) operator. Since trace is linear, we can just pass it right through:

\[ d(\|M\|^2) = d(\operatorname{tr}(M^T M)) = \operatorname{tr}(d(M^T M)). \]

Now, we need to solve for the derivative of \( d(M^T M) \). We simply use the same trick
when we differentiate \( x^2 \), but with a more concise notation:

\[ d(M^T M) = d(M^T) M + M^T dM = dM^T M + M^T dM. \]

Once again, \( d \) propagates through \( \cdot^T \) since the latter is linear. I hope that
now you have seen how powerful linearity is.

Note, however, that I'm not switching terms around due to the fact that matrix multiplication
is not commutative. When we use the fact that the derivative of a linear map is itself,
we must keep that linear map exactly the same.

However, we notice that everything is wrapped inside a trace operation. And fun fact:
\( \operatorname{tr}(A) = \operatorname{tr}(A^T) \). By pure coincidence, the two terms
in the final sum is actually each other's transpose!

\[ (M^T dM)^T = dM^T (M^T)^T = dM^T M. \]

Finally, we have:

\[
\begin{align*}
d(\|M\|^2) &= \operatorname{tr}(dM^T M + M^T dM) = \operatorname{tr}(dM^T M) + \operatorname{tr}(M^T dM)\\
&= 2\operatorname{tr}(M^T dM) = 2\operatorname{tr}(dM^T M).
\end{align*}
\]

There you go, that's the derivative of the squared Frobenius norm. It does not look like something
multiplied with \( dM \), but that's just a limitation of matrices: matrix multiplication
could only represent linear maps on vectors, not matrices. In fact, such a result could not even
exist (in non-trivial cases). If we can write \( d(\|M\|^2) = X d M \), then looking at the dimensions,
there is an immediate paradox. If \( M \) is \( m \times n \)-dimensional, then the matrix
multiplication must yield some \( \ldots \times n \)-dimensional matrix, and not a scalar!

Also, for those wondering that why aren't we finding the derivative for the Frobenius norm itself.
I have two reasons for that:
- It's more complicated due to the addition of the square-root operation.
- The Frobenius norm is usually used to describe some form of error. And minimizing that error
  is the same as minimizing the error squared. Hence, differentiating the squared Frobenius norm
  is a far more pleasant experience that does the exact same job.

## Conclusion

So that's it for the first part of this blog series. Without having to resort to indices, we
have developed a new theory for differentiation that works on all sort of vectors (not just
the usual \( \mathbb{R}^n \)). Also, I think this should be stated sooner, I just could not find
the space to, but this theory is based on that amazing lecture notes PDF. The total derivative
is kind of invented by me (it's probably the result of me looking at similar notations and
finally getting a sense of that, to be honest). In the process,
we have also discovered an ugly truth: not all derivatives can be represented using a clear and
concise form like the Jacobian matrix, as demonstrated above. And of course, treating those
derivatives directly as function objects while implementing NN algorithms would be a direct path
towards poor performance (I don't even think it's possible, considering the weights and whatnot).

Also, in this blog, there is a huge time skip. I learned about the Fréchet derivative while
I was a freshman (or even before), but I could only find the derivative of the squared Frobenius
norm just until recently (of course my freshman self could cheese it using indices, but you know
what I mean). I just came back to this problem after neglecting it for a long while,
after all.

In the next part, I think we will derive the math of Backpropagation on Multilayer Perceptrons.
It was yet another failed attempt of mine on training a handwritten digit recognizer.

![linearity and trace abuse](meme.png "(What is about to come in next parts)")
