---
layout: single
classes: wide
title: Neural Network Basics
date: 2021-09-14 14:02 -0500
author_profile: true
read_time: true
comments: false
share: true
usemathjax: true
---

Neural networks are everywhere today with applications like image recognition,
speech recognition, time series forecasting, and so on. In science, specially physics,
neural nets have been used to reduce the computational burden of several analyses like
candidate selection, noise subtraction etc. More recently, physics informed neural
networks have been used to solve differential equations as an alternative to finite
element analysis. Another application is in parameter estimation where the so called
likelihood-free inference techniques are an alternative (and a shift) from traditional
techniques of stochastic sampling like Monte-Carlo. But what is fundamental reason behind
neural nets working? I feel most internet resources dive into constructing a neural net
without mentioning the answer to this. I was curious about this (since I am taking up
a project on using neural nets), and I thought of writing this short article about it.

I should mention that while the term _neural network_ sounds fancy. Practically, the
understanding is pretty simple, and one is able to follow it with high-school math and
the knowledge of partial differentition.

## Practical working of a neural-net
Fundamental points:
- A neural net is basically a function - $f: \mathcal{R}^m \rightarrow \mathcal{R}^n$.
  Once trained, the outputs are deterministic (as opposed to stochastic).
  For simplicity, lets put $n=1$.
- It is formed out of composition of a) a linear transformation, followed by b) a non-linear
  activation function.
- The "deep" (not me. And I've heard that one many times) in deep-neural-nets is basically the
  total number of compositions, which are referred to as _hidden layers_. Let's consider
  the simplest example of a single hidden layer,
  
  $$ f = \beta\;\sigma\;\left(\sum_{j=1}^{m} W_j\;x_j + b\right). $$

  Here, $$W_j$$ are a set of parameters called _weights_ that are adjustable during the
  training. The parameter $$b$$ is called the bias. The function $$\sigma$$ is a non-linear
  activation function. An example is,
\begin{align}
  \sigma(x) = \left[1 + \exp(-x)\right]^{-1}.
\end{align}
  A class of activations are _squeezer_ function like the above - they take in any number and
  map it to the interval $$(0, 1)$$ (or something finite).
- If there are two hidden layers, the network becomes,
\begin{align}
  f = \beta\;\sigma^{(2)}\;\left(
          \mathbf{W}^{(2)}\cdot\sigma^{(1)}\left[
              \mathbf{W}^{(1)}\cdot\mathbf{X} + \mathbf{b}^{(1)}
          \right] + b^{(2)}
      \right).
\end{align}
  Note above that $$\mathbf{W^{(1)}}$$ is a matrix while $$\mathbf{W^{(2)}}$$ is a vector,
  and the "$$\cdot$$" is used both as a regular dot product and also a matrix multiplication
  operation.

## But why will such a composition work at all?
This was, at least for me, the more important question.

After all, the application of neural-nets lie in the practical difficulty that
we cannot know the relation between inputs and outputs in most practical
problems, and we depend on a neural net to do it for us, or at least approximate it
reasonably well.

Two papers - [Hornik et. al. (1990)](https://www.sciencedirect.com/science/article/abs/pii/0893608089900208)
and [Hornik (1991)](https://www.sciencedirect.com/science/article/abs/pii/089360809190009T) gives
the answer to this. Another paper that I found very cutely stated the result without being
super rigorous is [Kreinovich (1991)](https://www.sciencedirect.com/science/article/abs/pii/089360809190074F).

Now, unlike the previous section, following the content of these articles requires at least
some highlights/overview of real-analysis. The main idea is captured by the titles of the
mentioned papers: **neural networks are universal approximators**.

This means a neural network can be used to approximate any function! That is a _very_
powerful statement! Surely there must be some assumptions, right?
- I mean can it approximate _any_ function? Even weirdly/badly behaving ones?
  What about poles, singularities etc. etc. etc.?
- What are the constraints on what activation functions you can use?
- Does it work for any dimensional data?

Let's start with the last one since it is the easiest to answer - If a neural
net is the universal approximator for a one-dimensional, scalar function, the
extension to vector outputs is immediate since it's the same proof applied to
approximating the individual components of the vector-valued function.

In terms of activation functions, the _sufficient_ properties needed to work,
at least _in principle_, is
- Non-constant
- Bounded in domain considered

Most squeezer type functions definitely satisfy this condition. But note that
this does not preclude the _certain_ unbounded functions to be used, for example
$$\exp(x)$$, which can also be used as an activation. Just that the proof in
the above paper state that it is _sufficient_ (but not _necessary_) to be
bounded.

Finally, what functions can be approximated?
- Continuous (it can have kinks; non-differentiable)
- Bounded
- Defined on a compact subset of $$\mathcal{R}^m$$.

While the proofs in the papers above are daunting at first sight (maybe even
after), the overall message is relatively easy to understand. We all know
about _polynomial interpolation_ of a continuous function. The fact that
this works is the statement of the _Weierstrass theorem_ which states that
> each element of the set of continuous functions on a compact subset $$\mathbf{X}$$
of $$\mathcal{R}$$, denoted by $$C(\mathbf{X})$$, is well approximated by
a polynomial.

The proof that neural-networks are universal approximators essentially
delegates the proof to a generalization of the above, known as _Stone-Weierstrass_
theorem. A fact about polynomials is that multiplication of two polynomials
is another polynomial. This is a property satisfied by a mathematical structure
called an _algebra_. Now, 
> It can be shown that the set of all single-layer neural networks form an
_algebra_ i.e., composition of two nets is another net.

Now the statement of _Stone-Weierstrass_ (the way it makes most sense to me)
> An algebra of $$C(\mathbf{X})$$ is _dense_ in $$C(\mathbf{X})$$.

That's it! With the two statements above, the result that neural-nets are
universal approximators follow the same reason as polynomial approximation -
the set of neural-nets form an algebra, so they can approximate any
element of $$C(\mathbf{X})$$.

Now note that this is only a proof of existence. How one reaches the _right_
network i.e., how one adjusts the weights and biases, how many nodes to consider
in a layer, how many layers to use etc, is an entirely different problem.
And this is where the nature and intuition about a specific problem becomes
important, and is an active area of research in algoritms and computation.


I must mention that most of the statements above are the way I understand
the statements of the proof, and certainly leaves out a lot of rigor. But
it is sufficient for my understanding of the bottomline. Hope this helps
someone who was like me, wanting to find some fundamental reason why the
hype with neural-nets after all.
