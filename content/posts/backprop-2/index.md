+++
date = '2025-07-18T10:24:33+07:00'
title = 'backprop-2.md'
+++

# Finishing what I left off: deriving Deep Learning without touching a single index

After 8 months since the last post on deriving backpropagation, I finally got
some time to write the sequel. Well, during that 8 months,
[a](https://github.com/nrs-org/nrs-impl/discussions/539#discussioncomment-11666567)
[lot](https://github.com/nrs-org/nrs-impl/discussions/539#discussioncomment-11704056)
[has](https://github.com/nrs-org/nrs-impl/discussions/539#discussioncomment-11706695)
[happened](https://github.com/nrs-org/nrs-impl/discussions/625#discussioncomment-12690175),
but what matters is that I finally got some time to write this post.

In the last post, I defined derivatives on normed vector spaces, and
used that to find the derivative of one of the most important functions: the
Frobenius norm. The heart of our approach is using linearity to pass the
derivative operator through the functions like the trace function to end up with
a nice and clean expression for the derivative of complicated formulas.

Now, without further ado, let's try to apply what we have learned to derive
the formula for backproparation. But before that, we will work with a much more
simpler problem: Linear Regression.

## Gradient and traces

To do any sorts of gradient descent, we need to first define the gradient. Well,
the traditional definition of the gradient is a vector that points in the
direction of the steepest ascent of a function. However, the concept of "ascent"
and "descent" only exists for functions from a vector field\(V\) to an ordered field
like \(\mathbb{R}\), i.e. scalar-valued functions, since we can't really order
vectors in a meaningful way.

Intuitively, the gradient of a function \(f: V \to \mathbb{R}\) at a point \(x\)
is a vector \(u\) such that the directional derivative of \(f\) at \(x\) in the
direction of \(u\) is maximized, i.e. the rate of change of \(f\) at \(x\) in
the direction of \(u\) is the largest among all unit vectors in \(V\).
Representing this with the derivative operator \(L\), we can write this as:

**Definition**: A unit direction \(u\) with \(\|u\| = 1\) is a normalized gradient
vector of a function \(f\) at \(x\) if:
\[ L(u) \leq L(u'), \forall u' \in V: \|u'\| = 1, \]
where \(L\) denotes the derivative of \(f\) at \(x\) in the direction of \(u\).

The problem with normalized gradient vectors is that they are, well, normalized.
It does not encode the information of the rate of change, i.e. \(L(u)\), and it
is computationally more expensive to compute than the normal gradient.
Hence, a trivial way to fix this is to multiply the normalized gradient vector
by the rate of change, i.e. \(L(u)\), to get the gradient vector:

**Definition**: The product of the normalized gradient vector \(u\) of \(f\) at
\(x\) and the rate of change \(f'(x)(u)\) is called a gradient vector of a 
function \(f\) at \(x\).

We will denote the gradient vector of \(f\) at \(x\) as
\[ \nabla f(x) = L(u) u, \]
where \(u\) is a normalized gradient vector of \(f\) at \(x\).
However, we haven't proven that the gradient vector is unique, so this notation
might not make sense. We will still use this notation,
but we will not any reasoning that assumes that the uniqueness of the gradient.

Then, we will find a way to find a gradient vector from the derivative. We have
the following result:

**Theorem**: If \(f\) has derivative \(L(dX)=\operatorname{tr}(A^T dX)\), then
a gradient of \(f\) at \(X\) is given by:
\[ \nabla f(X) = A, \]
where the norm in use is the Frobenius norm.

We have two cases:

- If \(A\) is zero, then \(L = 0\). Then, any unit direction \(dX\) is a
  normalized gradient vector with \(L(dX) = 0\) and therefore zero is a gradient
  vector.
- If \(A\) is non-zero, then we can normalize it: \(\tilde{A} = \frac{A}{\|A\|}\).
  Then if \(\tilde{A}\) is a normalized gradient vector, then:
  \[ \tilde{A} L(\tilde{A}) = \tilde{A} \operatorname{tr}(A^T \tilde{A}) = A \]
  is a gradient vector of \(f\) at \(X\). To prove \(\tilde{A}\) is a normalized
  gradient vector, we need to show that:
  \[ L(\tilde{A}) \leq L(U), \forall U: \|U\| = 1. \]
  This is equivalent to:
  \[
  \operatorname{tr}(A^T \tilde{A}) \leq \operatorname{tr}(A^T U), \forall U: \|U\| = 1.
  \]
  "Unnormalizing", this is equivalent to:
  \[ \|A\|^2 = \operatorname{tr}(A^T A) \leq \operatorname{tr}(A^T U), \forall U:
  \|U\| = \|A\|. \]

  Then, one can see that this is equivalent to the Cauchy-Schwarz inequality.
  More specifically, \(\langle A, B\rangle = \operatorname{tr}(A^T B)\) is an
  inner product in the space of matrices, and by the Cauchy-Schwarz inequality:
  \[
  \langle A, U\rangle^2 \leq \langle A, A\rangle \langle U, U\rangle\\
  \Rightarrow |\operatorname{tr}(A^T U)| \leq \sqrt{\|A\|\|U\|} = \|A\|.
  \]

With this, we have found an explicit gradient formula for a relatively wide
range of functions, not just for vectors, but also for matrices.

## Deriving the formula for Linear Regression

Linear Regression is a simple problem, and we don't really need heavy machinery
to solve it. Hence, we will try to "speedrun" through it, just to see how our
approach works. We will start with the general case. We are given a training
dataset \(D \subseteq \mathbb{R}^n \times \mathbb{R}^m\), where each data point
is a pair \((x_i, y_i)\) where \(x_i \in \mathbb{R}^n\) is the input and \(y_i
\in \mathbb{R}^m\) is the output. We want to learn a linear function \(f:
\mathbb{R}^n \to \mathbb{R}^m\) such that the output of the function is as close
as possible to the output of the training data.

Traditional Machine Learning packs the dataset into two matrices: \(X \in
\mathbb{R}^{N \times n}\) and \(Y \in \mathbb{R}^{N \times m}\), where \(N = |D|\)
is
the number of data points, \(n\) is the number of input features, and \(m\) is
the number of output features. Each row of \(X\) is a data point's input, and
each row of \(Y\) is the corresponding output...

Which makes no sense.

They treat each data point, a vector, as a row vector, which most linear
algebraic notation consider it a **linear functional**, an element in the dual
space of the original vector space. While the weight matrix, representing the
function we try to seek, is treated as a vector. And they write things like:
\[ X w, \]
which means, apply \(w\) to the "function" \(X\)?????????? What is that supposed
to mean?

Of course, since I'm a contrarian who tries to study geometric algebra and
nonstandard calculus instead of the more traditional vector calculus and
standard calculus, I will try to write this in a more sensible way by
transposing everything. Pack the dataset into two matrices: \(X \in
\mathbb{R}^{n \times N}\) and \(Y \in \mathbb{R}^{m \times N}\), where each
column of \(X\) is a data point's input, and each column of \(Y\) is the
corresponding output. Then, we can write the linear function as a matrix
\[ W \in \mathbb{R}^{m \times n}, \]
and the linear function can be written as
\[ f(x) = W x. \]

Beautiful, right?

Now, since we want to learn \(f\), we need a metric to see how good an \(f\) is.
The most popular choice is the mean squared error loss function:
\[ \sum_{i = 1}^N \|y_i - f(x_i)\|_2^2, \]
which can be rewritten as
\[ \|Y - f(X)\|_2^2 = \|Y - W X\|_2^2, \]
where the norm \(\|\cdot\|\) denotes the Forobenius norm,
something we had worked with in the last post.

While, we are at it, let's add a regularization term to the loss function:
\[ l(W) = \|Y - W X\|_2^2 + \lambda \|W\|_2^2, \]
where \(\lambda\geq 0\) is a hyperparameter that controls the amount of
regularization. For the motivation of this term, you can read about Ridge
regression online, we are only focusing on the mathematical aspect of it.

Now, since the objective function is convex, solving this is equivalent to
solving \(l'(W) = 0\). Or if we want to use gradient descent, we need to compute
the gradient (derivative for us) of the loss function:

\[
\begin{align*}
dl &= d (\|Y - W X\|_2^2 + \lambda \|W\|_2^2)  \\
&= d (\|Y - W X\|_2^2) + \lambda d (\|W\|_2^2) \\
&= 2 \operatorname{tr} ((Y - W X)^T  d (Y - W X)) + 2 \lambda
\operatorname{tr}(W^T d W)\\
&= - 2 \operatorname{tr} ((Y - W X)^T  d W X) + 2 \lambda \operatorname{tr}(W^T
d W).
\end{align*}
\]
Using the commutativity of the trace, we can rewrite this as
\[
\begin{align*}
dl 
&= 2 \operatorname{tr} (X (W X - Y)^T  d W) + 2 \lambda \operatorname{tr}(W^T
d W)\\
&= 2 \operatorname{tr} ((X(WX-Y)^T + \lambda W^T) dW)\\
&= 2 \operatorname{tr} ((X(X^T W^T-Y^T) + \lambda W^T) dW)\\
&= 2 \operatorname{tr} ((XX^T W^T- XY^T + \lambda W^T) dW)\\
&= 2 \operatorname{tr} ((XX^T+\lambda I) W^T- XY^T) dW).
\end{align*}
\]

Hence, applying the theorem in the gradient section, we can see that
the gradient of \(l\) with relation to \(W\) is:
\[
\nabla l(W) = (XX^T+\lambda I) W^T- XY^T.
\]

Solving \(\nabla l(W) = 0\) gives,
\[
\begin{align*}
&(XX^T + \lambda I) W^T = XY^T \\
&\iff W (X X^T + \lambda) = YX^T \\
&\iff W = Y X^T (X X^T + \lambda)^{-1}.
\end{align*}
\]

If we convert this to the standard notation, we have:
\[
w = W^T = (X X^T + \lambda) X Y^T = (X'^T X' + \lambda)^{-1} X'^T Y',
\]
the exact formula for the weights of Linear Regression with Ridge regression,
where \(X' = X^T\) and \(Y' = Y^T\) are the input and output matrices
in standard notation, respectively. Hence, this confirms the validity of our approach.

## Deriving the formula for Backpropagation

Let's now try to derive the formula for backpropagation using the same
approach. First, I will describe a simple feedforward neural network (NN)
as follows:

**Definition**: A **feedforward neural network** is a function \(f: \mathbb{R}^n
\to \mathbb{R}^m\) defined as a composition of \(L\) layers, where each layer
is a function \(f_l: \mathbb{R}^{n_l} \to \mathbb{R}^{n_{l+1}}\) with \(n_0 =
n\)
and \(n_L = m\). Each layer is a linear transformation followed by a non-linear
activation function:
\[ f_l(x) = \sigma_l(W_l x + b_l), \]
where \(W_l \in \mathbb{R}^{n_{l+1} \times n_l}\) is the weight matrix of the
layer, \(b_l \in \mathbb{R}^{n_{l+1}}\) is the bias vector of the layer, and
\(\sigma_l: \mathbb{R}^{n_{l+1}} \to \mathbb{R}^{n_{l+1}}\) is the activation
function of the layer. The output of the NN is given by:
\[ f(x) = f_L \circ f_{L-1} \circ \ldots \circ f_1 (x). \]

First, we will solve for the derivative of the activation function, which is
usually a element-wise function. More specifically, if \(M\in\mathbb{R}^{m
\times n}\) is the input to the activation function \(\sigma\),
then the derivative of the activation function is simply
\[
\sigma(M) = \begin{bmatrix}
\sigma(M_{11}) & \sigma(M_{12}) & \ldots & \sigma(M_{1n}) \\
\sigma(M_{21}) & \sigma(M_{22}) & \ldots & \sigma(M_{2n}) \\
\vdots & \vdots & \ddots & \vdots \\
\sigma(M_{m1}) & \sigma(M_{m2}) & \ldots & \sigma(M_{mn})
\end{bmatrix}.
\]

This, unfortunately, does not have any linear-algebraic interpretation, so we
have to work with indices for now. However, we can derive the derivative of the
activation function as follows:

**Theorem**: If \(S = \sigma(M)\) where \(\sigma\) is a differentiable
activation function with derivative \(\sigma'\), then the derivative 
of the activation function \(\sigma\) for matrices is given by,
in differential notation,
\[
d (\sigma (M)) = \sigma'(M) \odot d M,
\]
where \(\sigma'\) denotes the scalar derivative of \(\sigma\), applied
element-wise to elements of \(M\), \(\odot\) denotes the Hadamard product
(element-wise product) of matrices.

To find the expression for \(dS\) above, we return to our good-old limit:
\[\lim_{d M \to 0} \frac{\|\sigma(M + d M) - \sigma(M) - L(d M)\|}{\|d M\|} = 0 \]
Now, we let \(d M\) approaches 0 by fixing every element except \(M_{i j}\),
then, naturally, we would expect:
\[\lim_{t \to 0} \frac{\|\sigma(M_{i j} + t) - \sigma(M_{ij}) - L(t E_{ij})_{ij}\|}{\|t\|} = 0, \]
and
\[\lim_{t \to 0} \frac{\|\sigma(M_{i' j'}) - \sigma(M_{i'j'}) - L(t E_{ij})_{i'j'}\|}{\|t\|} = 0, \]
where \(E_{ij}\) denotes the matrix with 1 at position \((i, j)\) and 0
elsewhere, for any \((i, j) \neq (i', j')\).

The second expression implies \(L(E_{ij})_{i'j'} = 0\), while the first implies
\(L(E_{ij}) = \sigma'(M_{ij})\). Hence, \(L(E_{ij}) = \sigma'(M_{ij}) E_{ij}\).
Then, for every \(dM\), represent it as a linear combination of the basis
matrices \(E_{ij}\) to have the desired result.

Finally, since we started from the limit, we have to recheck that the
derivative we found is actually the derivative. This can be easily verified.

With this, we can finally derieve the formula for backpropagation. First, define
\[ a_0 = x, z_l = W_l a_{l - 1} + b_l, a_l = \sigma(z_l), \]
as immediate values of the NN, where \(a_l\) is the output of the \(l\)-th
layer, \(z_l\) is the pre-activation value of the \(l\)-th layer, and
\(W_l\) and \(b_l\) are the weight matrix and bias vector of the \(l\)-th layer,
as defined above. It follows that:
\[
d a_l = d (\sigma (z_l)) = \sigma'(z_l) \odot d z_l,\\
d z_l = d (W_l a_{l - 1} + b_l) = d W_l a_{l - 1} + W_l d a_{l - 1} + d b_l.
\]

Now, if we are interested in \(dW_l\) and \(db_l\), i.e. the derivatives with
respect to \(W_l\) and \(b_l\), then we would do:
- Set \(d b_l = 0\) and \(d a_{l - 1} = 0\) to get: \(d a_l = \sigma'(z_l) \odot
dW_l a_{l-1}\).
- Set \(d W_l = 0\) and \(d a_{l - 1} = 0\) to get: \(d a_l = \sigma'(z_l) \odot
d b_l\).

Now, we will do a funny trick: we know that \(\sigma'(z_l)\) are vectors, and
the Hadamard product of two vectors \(u\) and \(v\) can be written as:
\(u \odot v = \operatorname{diag}(u) v\), where \(\operatorname{diag}(u)\) is
the diagonal matrix with \(u\) on the diagonal. Hence, we can rewrite everything
with only matrix multiplications. For brevity, denote \(A_l = \operatorname{diag}(\sigma'(z_l))\).

Then, we can rewrite the above equations as:
\[d a_l = A_l d W_l a_{l - 1}\]
and
\[d a_l = A_l d b_l.\]
Now, if we apply this multiple times, we would have:
\[ d a_L = A_L W_L A_{L -1} W_{L - 1} \ldots A_k (dW_l a_{l-1} + db_l). \]

Now, we define a loss function \(C\) for the neural network, with derivative
\(C'\) such that:
\[ d C = C' d a_L, \]
then we can denote the huge product on the left:
\[\Delta_L = C' A_L, \Delta_l = \Delta_{l+1} W_{l+1} A_l\]
to have
\[ d C = \Delta_l (d W_l a_{k-1} + d b_l). \]
With this, we can finally find the gradient of the loss function with respect to
the weights and biases of the neural network:
\[ d C
= \Delta_l (d W_l a_{l-1} + d b_l)
= \operatorname{tr}(\Delta_l (d W_l a_{l-1} + d b_l))\\
= \operatorname{tr}(a_{l-1}\Delta_l d W_l) + \operatorname{tr}(\Delta_l d b_l).
\]
So,
\[
\nabla_{W_l} C = \Delta_l^T a_{l-1}^T, \quad \nabla_{b_l} C = \Delta_l^T.
\]

Now, we will transform this to the traditional notation. We will denote
\[ \delta_k = \Delta_k^T, \]
then:
\[
\delta_L = A_L^T C'^T = \operatorname{diag}(\sigma'(z_L)) C'^T = \sigma'(z_L)
\odot \nabla_{a_L} C,\\
\]
and
\[
\delta_l = A_l^T W_{l + 1}^T \Delta_{l+1}^T =
\operatorname{diag}(\sigma'(z_l)) W_{l + 1}^T
\delta_{l + 1} = \sigma'(z_l) \odot W_{l + 1}^T \delta_l.
\]
Meanwhile,
\[
\nabla_{W_l} C = \delta_l a_{l-1}^T, \quad \nabla_{b_l} C = \delta_l.
\]
which is consistent with the result in the
[Wikipedia page](https://en.wikipedia.org/wiki/Backpropagation#Matrix_multiplication).
