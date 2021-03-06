<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# A Brief Introduction to Wasserstein GANs

This blog is written to offer an __intuitive__ interpretation of the mathematical insights in the well known paper [Wasserstein GANs(WGANs)](https://arxiv.org/pdf/1701.07875.pdf). In 2014, a new deep learning framework for generative models: [Generative Adversarial Nets(GANs)](https://arxiv.org/pdf/1406.2661.pdf) was introduced with great success. However, unlike other deep generative models (e.g., Variational AutoEncoder), GANs are notoriously found to be **unsuccessful**(the generator generate nothing but garbage), **unstable**(the training losses do not converge), and suffers from the **mode collapse**(the generator fails to generative diverse samples). To this end, WGAN, a new GAN framework is invented. 

We will go through 3 papers step-by-step and you will eventually take a tumble to the bottom of GAN.

- [Generative Adversarial Nets(GANs)](https://arxiv.org/pdf/1406.2661.pdf)

- [Towards Principled Methods for Training Generative Adversarial Networks](https://arxiv.org/pdf/1701.04862.pdf)

- [Wasserstein GANs](https://arxiv.org/pdf/1701.07875.pdf)


## Generative Adversarial Nets

### Introduction

GANs introduces a novel unsupervised training framework for generative models: adversarial learning. Given some "real" samples(say images), it simultaneously trains a __generator(G)__ to generate "fake" samples, and a __discriminator(D)__ to distinguish the "real" ones from "fake" ones. In the end, **G** will generate samples so real that **D** is unable to discriminate against them. 

### Objective Function and its Mathematical Intuition:

The objective function of GAN is:

$$ V(G, D) = \underset{G}{\min} \underset{D}{\max} \underset{x \sim \mathbb{P}_r}{\mathbb{E}}[\log D(x)] + \underset{z \sim \mathbb{P}}{\mathbb{E}}[\log (1-D(G(z)))] $$,

where $$z$$ is a "genesis" low-dimensional vector (e.g., sampled from a normal or uniform distribution), $$G(z)$$ is a generator (or decoder) network that generates a real sample from the "genesis" (e.g., through deconvolution to generate 256x256 images), and $$D(x)$$ is a discriminator network that represents the probability that our input samples(images) $$x$$ came from the real data rather than generative data, which indicates that: $$D(x) \in [0,1]$$. (hanwang: more explain about what is 0 and 1, say it directly)

During the training of discriminator networks $$D$$, we want the discriminator to accept real data and reject generated data, thus for real samples $$x \sim \mathbb{P}_r$$ and generated samples $$G(Z),z \sim \mathbb{P}$$, we want $$D(x)$$ to be large and $$D(G(z))$$ to be small. Thus, the objective function $$V(G, D)$$ suits this cases well: by increasing $$D(x)$$ and decreasing $$D(G(z))$$, we are actually increasing the objective function. On the contrary, while training generator networks $$G$$, we want our discriminator $$D$$ makes mistakes thus decreasing the objective function. 

Please tell that GAN can be considered as a data-driven metric learning problem. In this way, it would be helpful to understand why we need to redifine the distance using W-metric. from fully hand-crafted norm, to Mahanlonobis distance, to GAN. 



[This site](https://sigmoidal.io/beginners-review-of-gan-architectures/) will give you more information about the network architecture of GANs.

### Training Algorithm and Theoretical Results:

The training algorithm for GANs from the [GANs Paper](https://arxiv.org/pdf/1406.2661.pdf) is show below:

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/GAN_Training_Algorithm.png?raw=true" width="600">
</p>

- __Analysis of Discriminator__

For given generator $$G$$, we can substitute $$D(G(z))$$ with $$D(x)$$, because for any noise $$z$$ in the prior $$\mathbb{P}$$, $$G(z)$$ will generate a data sample(image). So the optimal discriminator $$D$$ will maximize our objective function(assuming the density functions are continuous):

$$V(G,D)=\int_xP_r(x)\log(D(x))+P_g(x)\log(1-D(x))dx$$ 

And by solving the gradient with respect to $$D(x)$$, we know that the optimal discriminator is $$D^*_G(x)=\frac{P_r(x)}{P_r(x)+P_g(x)}$$.

- __Analysis of Generator__

According to the training algorithm, we start training our generator when our discriminator is welled trained, ideally, our discriminator $$D^*_G(x)=\frac{P_r(x)}{P_r(x)+P_g(x)}$$. So during the training of generator, we want to minimize our objective function

$$C(G)=\underset{x \sim \mathbb{P}_r}{\mathbb{E}}[\log \frac{P_r(x)}{P_r(x)+P_g(x)}] + \underset{x \sim \mathbb{P}_g}{\mathbb{E}}[\log \frac{P_g(x)}{P_r(x)+P_g(x)}]$$

Using some trick in [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) and [Jensen–Shannon divergence](https://en.wikipedia.org/wiki/Jensen%E2%80%93Shannon_divergence), our objective function for $$G$$ can be written in this way: 
$$C(G)=-\log4 + 2JSD(\mathbb{P}_r||\mathbb{P}_g)$$. By the quality of JS-divergence, we know that: $$JSD(\mathbb{P}||\mathbb{Q})\in[0,\log2]$$. So ideally, when the objective function $$C(G)$$ reaches its minimum, we have $$JSD(\mathbb{P}_r||\mathbb{P}_g)=0$$, which indicates that $$P_r(x)=P_g(x)$$ [almost everywhere](https://en.wikipedia.org/wiki/Almost_everywhere). 

- __The -log Alternative__

In real case, during the training of generator $$G$$, people found out that the cost of generator does not decrease after using SGD, the [GAN tutorial(section 3.2)](https://arxiv.org/pdf/1701.00160.pdf) claims this problem is caused by a saturated cost function of generator. Thus, the tutorial uses another cost function of $$-\log(x)$$ instead of $$\log(1-x)$$, that is, instead of minimizing $$C_1(G)=\underset{z \sim P}{\mathbb{E}}[\log (1-D(G(z)))]$$, we minimize $$C_2(G)=\underset{z \sim P}{\mathbb{E}}[-\log (D(G(z))]$$. The difference between these two functions are shown in the following picture:

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/-log_alternative.png?raw=true" width="600">
</p>

Where the blue curve represents $$y=-\log(x)$$ and the the red curve represents $$y=\log(1-x)$$. Since our generator $$G$$ is updated after our discriminator $$D$$ is well trained, thus our discriminator will be stronger than our generator and $$D(G(z))$$ will be very small, say close to 0. In this case, we can learn from the picture above that the gradient of the red curve is much flatter than the blue curve, which means that SGD will have less effect on the first cost function $$C_1(G)$$. And thus, using $$C_2(G)$$ as an alternative seems to be a wiser choice in this case. 

It seems that by this minmax training process, we will have a generated distribution $$\mathbb{P}_g$$ that is equal to our real distribution $$\mathbb{P}_r$$ almost everywhere, so by playing this minmax game until equilibria, our goal of generating 'authentic' data is achieved. Sadly, this problem is still far from closed.

### Problems in Traditional GANs:

During the training of traditional GANs, we will frequently encounter these three problems: __generator failure__, __instability__, and __mode collapsing__

- __Generator Failure__

Not all training of GANs will finally generate meaningful results, what sometimes happens is that while the discriminator gets better during training, generator will fail and eventually generate garbage(source: [WGANs paper Figure 12](https://arxiv.org/pdf/1701.07875.pdf)):

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Generator_faliure.png?raw=true" width="600">
</p>

- __Instability__

During the training, we frequently found that our generator loss and its variance are increasing, even when their generated samples are getting better(source: [WGANs paper Figure 8](https://arxiv.org/pdf/1701.07875.pdf)):
<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Unstable_generator.png?raw=true" width="600">
</p>

- __Mode Collapsing__

Mode collapsing means that our generator fails to generate various data samples, instead, it 'collapses' into some fix samples(source [WGANs paper Figure 14](https://arxiv.org/pdf/1701.07875.pdf)):
<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Mode_collapse.png?raw=true" width="600">
</p>

From the picture above, although we randomly choose 64 $$z$$ from our prior, many generated results collapse into few images.

## Towards Principled Methods for Training Generative Adversarial Networks

### Introduction

Since the original GANs suffers from __generator failure__, __instability__, and __mode collapsing__, this paper provides rigious proof to say why previous GANs will eventually encouter those two issues and provides a __better cost function(or a better metric to evaluate the 'similarity' between two probability distributions)__ to avoid these issues. 

### The reasons for generator failure in GANs

In real cases, we can prove that there is always a **perfect** discriminator $$D^*(x)$$ that can perfectly distinguish real data from generated data, and gradient descend method **has no effect** on this discriminator $$D^*(x)$$, which explains why our __discriminator gets better and our generator fails__ during training.

- __Perfect Discriminator Theorem([Section 2.1](https://arxiv.org/pdf/1701.04862.pdf))__

Assume that the [supports](https://en.wikipedia.org/wiki/Support_(mathematics)) of our real sample distribution $$\mathbb{P}_r$$ and our generated sample distribution $$\mathbb{P}_g$$ are [submanifolds](https://en.wikipedia.org/wiki/Submanifold) $$\mathcal{M}$$ and $$\mathcal{P}$$ in our feature space $$\mathcal{X}$$(the vector space of final fully connected layer in the discriminator network). Then we can always find a optimal discriminator $$D(x)\rightarrow[0,1]$$, s.t. $$D(x)\mid_{x\in\mathcal{M}}=1, D(x)\mid_{x\in\mathcal{P}}=0$$ and $$\nabla_xD(x)\mid_{x\in\mathcal{M}\cup\mathcal{P}}=0$$ [almost everywhere](https://en.wikipedia.org/wiki/Almost_everywhere).

To intuitively understand this theorem, we can divide this problem in two parts: $$\mathcal{M}\cap\mathcal{P}=\emptyset$$ and $$\mathcal{M}\cap\mathcal{P}\neq\emptyset$$ 

(a) When: $$\mathcal{M}\cap\mathcal{P}=\emptyset$$ 

This means that the intersect between the supports of $$\mathbb{P}_r$$ and $$\mathbb{P}_g$$ is empty. In this cases, the following picture will provide some intuitive explanations(for detailed proof, check theorem 2.1 in [this paper](https://arxiv.org/pdf/1701.04862.pdf)):

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Perfect_descriminator_below.png?raw=true" width="480">
</p>

In the picture above, assume that $$\mathcal{M}$$ is the red submanifold and $$\mathcal{P}$$ is the green submanifold, both of them are 2 dimensional manifolds in 3 dimensional space. An obvious optimal discriminator will be a sigmiod like surface, which classifies all points above the blue manifold with true and below blue manifold with fake. An interesting attribute of sigmoid like function is that it suffers from saturated gradients, in the picture above, we can learn that this discriminator(blue manifold) can perfectly discriminate these two manifolds and gradient descend does not work on the supports of the $$\mathcal{M}$$(green manifold) and $$\mathcal{P}$$(red manifold).

(b) When: $$\mathcal{M}\cap\mathcal{P}\neq\emptyset$$

To understand the proof in this case, we have to introduce a mathematical idea of [transversal intersection](http://mathworld.wolfram.com/TransversalIntersection.html) and [perfectly aligned(definition 2.2)](https://arxiv.org/pdf/1701.04862.pdf) between two manifolds. If you don't understand the math behind these two idea, it is totally fine, these following pictures will give you some idea about perfect aligned manifolds and not perfectly aligned manifolds:

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/perfectly_align.png?raw=true" width="480">
</p>

- __two perfectly aligned circles in 3 dimensional space, their intersection is a oval like shape(2 dimensional manifold)__

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/not_perfectly_align.png?raw=true" width="480">
</p>

- __two not perfectly aligned circles in 3 dimensional space, their intersection is a line segment(1 dimensional manifold)__

<p align="center">
  <img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Not_perfectly_align_gaussian.png?raw=true" width="480">
</p>

- __two not perfectly aligned gaussian spheres in 3 dimensional space, their intersection is a curve between the green and red gaussian distribution(1 dimensional manifold)__

From the previous pictures we can have some intuitive concepts about __perfectly aligned__: when two manifolds $$\mathcal{M}$$ and $$\mathcal{P}$$ are not perfectly aligned, the [measure](https://en.wikipedia.org/wiki/Measure_(mathematics)) of the $$\mathcal{M}\cap\mathcal{P}$$ intersect strictly less than the measure of both $$\mathcal{M}$$ and $$\mathcal{P}$$, which means that $$\mathcal{M}\cap\mathcal{P}$$ has __0__ measure on $$\mathcal{M}$$ and $$\mathcal{P}$$. Speaking in a intuitive way, when two intersected manifolds are __not perfect aligned__, their intersection area are very small(comparing with their original size).

Also, [lemma 2](https://arxiv.org/pdf/1701.04862.pdf) tells us that in real case, the probability that two random distributions are perfectly aligned are actually __extremely small(equals to 0)__. Here are some pictures to intuitively explain this lemma:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Nearly_perfectly_align_gaussian.png?raw=true" width="480">
</p>

- __two nearly perfectly aligned gaussian sphere(shift red manifold -0.01 to the left and the greed manifold by +0.01 to the right), their intersection is still a curve(1 dimensional manifold)__

The picture above tells us one thing about this lemma: if we randomly put 2 manifolds of gaussian distributions into a 3 dimensional space, these two manifolds are perfectly aligned if and only if the parameters(mean vector and covariance matrix) are the same. Intuitively speaking, this probability is extremely small.

Well here is another vivid yet a little bit bloody example: imagine that every person is a 3 dimensional manifold in our 4 dimensional space(time is added as another dimension). If all people are randomly distributed in this 4 dimensional space, in most cases, given 2 people, they are disjoint. 

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Two_men.png?raw=true" width="360">
</p>

Well sometimes, there is some probability when the manifolds of two people intersect but not perfectly aligned with each other(this situation happens when these 2 guys are having body contact, e.g. hand handshaking, hugging, etc.). 

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Handshake_two_men.png?raw=true" width="360">
</p>

However, the situation that the two manifolds of these two people are perfectly aligned is when __a part of one guy's body is inside the other's body__, which is extremely rare in real cases. This may sound very bloody, but it will give you some intuition why the manifolds of two random probability distribution barely perfectly aligned.

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Bloody_two_men.png?raw=true" width="360">
</p>

Here, we can safely say that even when the manifolds $$\mathcal{M},\mathcal{P}$$ of our two distribution $$\mathbb{P}_r,\mathbb{P}_g$$ are not disjoint, their intersection area $$\mathcal{M}\cap\mathcal{P}$$ is very small with respect to their own size(have 0 measure on manifold $$\mathcal{M}$$ and $$\mathcal{P}$$). 

Also, we can denote $$\mathcal{L} = \mathcal{M} \cap \mathcal{P}$$ as the intersect of $$\mathcal{M}$$ and $$\mathcal{P}$$, $$\tilde{\mathcal{M}} = \mathcal{M}\backslash\mathcal{L}$$ and $$\tilde{\mathcal{P}} = \mathcal{P}\backslash\mathcal{L}$$. From the previous proof, we know that the measure of $$\mathcal{L}$$ is 0 with respect to $$\mathcal{M}$$ and $$\mathcal{P}$$. Also according to the definition of $$\tilde{\mathcal{M}}$$ and $$\tilde{\mathcal{P}}$$, we know that: $$\tilde{\mathcal{M}}\cap\tilde{\mathcal{P}}=\emptyset$$.

Guess what, right now we have two disjoint manifolds($$\tilde{\mathcal{M}}$$ and $$\tilde{\mathcal{P}}$$) again! And by the same process from part (a), we can still find a optimal discriminator $$D^*(x)$$, s.t. $$D^*(x)$$ can perfectly discriminate $$\tilde{\mathcal{M}}$$ and $$\tilde{\mathcal{P}}$$. Well, what about the intersection $$\mathcal{L}$$? Well, recall the definition of [almost everywhere](https://en.wikipedia.org/wiki/Almost_everywhere) we mentioned before, we actually don't care about the classification on $$\mathcal{L}$$, because the size(measure) of $$\mathcal{L}$$ is too small with respect to $$\mathcal{M}$$ and $$\mathcal{P}$$.

Okay, at this point, we have intuitively went through the proof of the __perfectly discriminator theorem__. And this theorem explains why the traditional way of training GANs will sometimes encounter generator failure.


### The source of instability training loss in GANs 

Recall the two cost functions of our generator in the __-log alternative__ section: 

$$C_1(G)=\underset{z \sim P}{\mathbb{E}}[\log (1-D(G(z)))]$$

$$C_2(G)=\underset{z \sim P}{\mathbb{E}}[-\log (D(G(z))]$$

Actually these two cost functions suffer from some issues. 

- __Vanishing Gradient on $$C_1$$__

If we use $$C_1(G)$$ as our cost function for discriminator, [theorem 2.4](https://arxiv.org/pdf/1701.04862.pdf) can provide a rigorous proof that the gradient of $$C_1(G)$$ will vanish if we train our discriminator $$D$$ until convergence. And since experiments have shown this result, people often use $$C_2(G)$$ instead of $$C_1(G)$$ as the generator cost.

- __Unstable gradient on $$C_2$$__

However, $$C_2(G)$$ accounts for the instability in training. The proof of its instability is introduced in this theorem from [the paper](https://arxiv.org/pdf/1701.04862.pdf), as before, I will briefly go through this theorem and make it more understandable:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Theorem2.6.png?raw=true" width="600">
</p>

Using $$C_2(G)$$ as cost function for generator, when we train our discriminator until convergence, these things will happen:

(a) Assume that the difference $$\epsilon=D^*-D$$ between our discriminator and the optimal discriminator is a centered Gaussian distribution, also the difference $$r=\nabla_xD^*-\nabla_xD$$ between the gradient of our discriminator and the optimal discriminator is another Gaussian distribution which is independent with $$\epsilon$$. 

(b) The expectation of $$C_2(G)$$'s gradient equals to the expectation of a [Cauchy distribution](https://en.wikipedia.org/wiki/Cauchy_distribution). This happens because the [ratio distribution of two Gaussian distributions](http://mathworld.wolfram.com/NormalRatioDistribution.html) is a Cauchy distribution. 

(c) The [mean](https://en.wikipedia.org/wiki/Cauchy_distribution#Mean) and [variance](https://en.wikipedia.org/wiki/Cauchy_distribution#Higher_moments) of Cauchy distribution are undefined(infinity).

The following pictures(figure 3 from [the paper](https://arxiv.org/pdf/1701.04862.pdf)) will provides some evidence of increasing generator cost and generator variance:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/increasing_gradient.png?raw=true" width="600">
</p>

Now we have went through the reasons for unstable generator loss during training in traditional GAN.

### The reasons for mode collapsing in GANs

The goal of generator is to generator a distribution $$\mathbb{P}_g$$ that is very similar with our real data distribution $$\mathbb{P}_r$$, thus the cost function should reflect the similarity between our two distributions and minimizing the cost function will make our distributions more 'similar'. Ian Goodfellow mentions in his [tutorial on GANs(figure 14)](https://arxiv.org/pdf/1701.00160.pdf) that either KL divergence based or reverse KL divergence based cost function will provide bad generated results. 

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/KL:ReverseKL.png?raw=true" width="600">
</p>

The argument from the picture above indicates one thing: if we use $$KL(\mathbb{P}_r\parallel\mathbb{P}_g)$$ as the generator's cost function, our model will fit an average of the real data thus __generating fake pictures__ and if we use $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$ instead, our model will fit into part of the Gaussian mixture __incurring mode collapsing__. For detailed explanation, please refer to the introduction of [this paper](https://arxiv.org/pdf/1701.04862.pdf). At this point, you may find it confusing: the cost function of our generator is a $$JSD(\mathbb{P}_r\parallel\mathbb{P}_g)$$, which is an 'average' of $$KL(\mathbb{P}_r\parallel\mathbb{P}_g)$$ and $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$. Then why can't we balance this trade-off between fake results and mode collapsing?

The truth is, if we optimize our generator with the cost function $$C_1(G)$$, then we are indeed decreasing $$JSD(\mathbb{P}_r\parallel\mathbb{P}_g)$$. However, in real case we are actually optimizing $$C_2(G)$$. And there is an interesting [theorem(theorem 2.5)](https://arxiv.org/pdf/1701.04862.pdf), indicating that minimizing $$C_2(G)$$ using SGD is actually minimizing a $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$ based cost function.

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Theorem2.5.png?raw=true" width="600">
</p>

From the picture above, we can see that the expectation $$C_2(G)$$'s gradient is actually the gradient of $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)-2JSD(\mathbb{P}_g\parallel\mathbb{P}_r)$$, which indicates that we are actually decreasing a $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$ based cost function since $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)\in[0,\infty]$$ and $$JSD(\mathbb{P}_g\parallel\mathbb{P}_r)\in[0,\log2]$$. As we mention before, use $$KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$ based cost function will result in __mode collapsing__.

### A better metric for measuring the similarity between two probability distribution

- __Why KL/JS divergence are not good metric__

The key idea of training a generator is to __train a distribution that is as similar as possible to our real data distribution__. To achieve this goal, we will need a metric to __reflect the 'similarity' between our generator's distribution and our real data's distribution__. Ideally, this metric should some how reflect the 'distance' between two distributions, which means that if when two distribution __get more similar__, this metric should __have smaller value__. However, JS divergence and Kl divergence don't have this kind of quality. To prove this, recall the perfect discriminator theorem we mentioned before, we mentioned that $$\mathbb{P}_r$$ and $$\mathbb{P}_g$$ don't perfectly align. And by [this theorem(theorem 2.3)](https://arxiv.org/pdf/1701.04862.pdf), we know that as long as our two probability distribution $$\mathbb{P}_r$$ and $$\mathbb{P}_g$$ are not identical, $$KL(\mathbb{P}_g\parallel\mathbb{P}_r), KL(\mathbb{P}_g\parallel\mathbb{P}_r)$$ will max out.

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Theorem_2.3.png?raw=true" width="600">
</p>

- _Proof:_
<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/KL.png?raw=true" width="360">
</p>

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/JSD.png?raw=true" width="450">
</p>

Noticed that the measure of $$\mathcal{L}$$ is 0 on $$\mathcal{M}$$ and $$\mathcal{P}$$, thus the intergral $$\underset{\mathcal{L}}{\int} P_r\log \frac{P_r}{P_g}dx$$ and $$\underset{\mathcal{L}}{\int} P_r\log\frac{P_r}{\frac{1}{2}(P_r+P_g)}dx$$ equal to 0. Also, since $$\tilde{\mathcal{M}}$$ and $$\tilde{\mathcal{P}}$$ disjoint support of $$\mathbb{P}_r$$ and $$\mathbb{P}_g$$, then we know that $$P_r(x)\mid_{x\in\tilde{\mathcal{P}}}=0$$ and $$P_g(x)\mid_{x\in\tilde{\mathcal{M}}}=0$$.

And thus, we know that KL divergence and JS divergence are not good metric for measuring the 'similarity' between two distributions, because they __cannot reflect any improvement between two distributions when they are not perfectly aligned__, which is frequently happening in real case. 

- __A better metric: Wasserstein distance__

The wasserstein distance between two probability distribution can be defined in this way:

$$W(\mathbb{P},\mathbb{Q}) = \underset{\gamma\in\Gamma}{\inf}\int_{\mathcal{X}\times\mathcal{X}}||x-y||_2d\gamma(x,y)$$

Where $$\gamma$$ is the set of all possible joint probability distribution on $$\mathcal{X}\times\mathcal{X}$$ that have marginal on $$\mathbb{P}$$ and $$\mathbb{Q}$$. Don't be scared by this definition, this definition can be interpreted in this way: image the supports of probability distributions $$\mathbb{P}_r$$ and $$\mathbb{P}_g$$ are $$\mathcal{M}$$ and $$\mathcal{P}$$, we want to move all the points from one manifold $$\mathcal{P}$$ to construct manifold $$\mathcal{M}$$, and Wasserstein metric represents the __minimum__ total travel distance we have to move for __all the points__. [The third paper](https://arxiv.org/pdf/1701.07875.pdf) will provide an interesting example to compare the Wasserstein distance with KL/JS divergence. Also, [theorem 3.3](https://arxiv.org/pdf/1701.04862.pdf) can rigorously prove why Wasserstein distance is indeed a good metric, but that's not quite intuitive, so I did not include that part in this blog.


## Wasserstein GAN

### An example to compare KL/JS divergence with Wasserstein distance

Here we introduce an interesting example([Example 1](https://arxiv.org/pdf/1701.07875.pdf)):

Let $$Z\sim U[0,1]$$ be the uniform distribution on unit interval. Let $$\mathbb{P}_0$$ be the distribution of $$(0,Z)\in\mathbb{R}^2$$ and $$\mathbb{P}_\theta$$ be distribution $$(\theta,Z)\in\mathbb{R}^2$$. To be concise, the density function of $$\mathbb{P}_r$$ is: $$P_0(x,y)=1,(x=0,y\in[0,1]),P_0(x,y)=0,(\text{otherwise})$$ and the density function of $$\mathbb{P}_g$$ is: $$P_\theta(x,y)=1,(x=\theta,y\in[0,1]),P_\theta(x,y)=0,(\text{otherwise})$$. And here is a graph([From an interesting blog](https://www.alexirpan.com/2017/02/22/wasserstein-gan.html)) illustrate the case where $$\theta=1$$.

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Wgan_pic.png?raw=true" width="450">
</p>

Obviously, in this picture, we can learn that:

(a) When $$\theta\neq0$$

$$KL(\mathbb{P}_0\parallel\mathbb{P}_\theta)=KL(\mathbb{P}_\theta\parallel\mathbb{P}_0)=\infty$$ and $$JSD(\mathbb{P}_0\parallel\mathbb{P}_\theta)=\log2$$

(b) When $$\theta=0$$

$$KL(\mathbb{P}_0\parallel\mathbb{P}_\theta)=KL(\mathbb{P}_\theta\parallel\mathbb{P}_0)=0$$ and $$JSD(\mathbb{P}_0\parallel\mathbb{P}_\theta)=0$$

As for Wasserstein distance in this case, remember the idea of __moving points from one distribution to construct the other__, here we want to find a joint distribution of $$\mathbb{P}_0$$ and $$\mathbb{P}_\theta$$, s.t. for any $$\theta$$, this distribution will have minimum total moving distance for all the points. 

So intuitively, for this specific case, the minimum total moving distance here is to move all the points from $$\mathbb{P}_\theta$$ horizontally to the distribution $$\mathbb{P}_0$$, that is, find a mapping that map $$(\theta,y)$$ to $$(0,y)$$. 

Now let's look at the Wasserstein distance again:

$$W(\mathbb{P},\mathbb{Q}) = \underset{\gamma\in\Gamma}{\inf}\int_{\mathcal{X}\times\mathcal{X}}||x-y||_2d\gamma(x,y)$$

The joint distribution with minimum total moving distance for $$\mathbb{P}_0$$ and $$\mathbb{P}_\theta$$ is: $$P((x_0,y_0),(x_\theta,y_\theta))=\delta(\text{when} x_0=0,x_\theta=\theta,y_0=y_\theta\in[0,1])$$ and $$P((x_0,y_0),(x_\theta,y_\theta))=0,\text{(otherwise)}$$. Here $$\delta$$ is a positive constant that keep the integral of this density distribution equals to 1.

Thus, we can learn that the Wasserstein distance of $$\mathbb{P}_0$$ and $$\mathbb{P}_\theta$$ is $$\theta$$. From this example, we know that KL/JS divergence cannot serve as a good metric because they can only tell whether $$\mathbb{P}_0$$ and $$\mathbb{P}_\theta$$ are identical while Wasserstein distance can actually tell how 'close' $$\mathbb{P}_0$$ and $$\mathbb{P}_\theta$$ is. 

The following picture illustrate the value of JS divergence and Wasserstein metric while $\theta$ is changing:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/Wgan_pic1.png?raw=true" width="600">
</p>

### Wasserstein GAN in application

Okay, now we learn that Wasserstein metric is indeed better than KL/JS divergence, but finding a joint distribution $$\gamma$$ that minimize $$W(\mathbb{P},\mathbb{Q}) = \int_{\mathcal{X}\times\mathcal{X}}\parallel x-y\parallel_2d\gamma(x,y)$$ is really intractable. But, thanks to the [Kantorovich-Rubinstein duality](https://en.wikipedia.org/wiki/Wasserstein_metric#Dual_representation_of_W1), we can actually change this form into a doable way: 

$$W(\mathbb{P}_r,\mathbb{P}_g) = \underset{\parallel f\parallel_L\leq1}{\sup}\mathbb{E}_{x\sim\mathbb{P}_r}[f(x)]-\mathbb{E}_{x\sim\mathbb{P}_g}[f(x)]$$

Where $$\parallel f\parallel_L\leq k$$ is the [k-Lipschitz constraints](https://en.wikipedia.org/wiki/Lipschitz_continuity):

$$f:\mathbb{R}^d\rightarrow\mathbb{R} \text{ s.t. } \parallel f(x_1)-f(x_2)\parallel_2\leq k \parallel x_1-x_2\parallel_2$$

Now the only unknown thing in this formula is the Lipschitz function $$f$$, this paper calls $$f$$ as 'critic' and uses a weight clipping method to parameterize the weight in $$f$$, but later on, a [new paper](https://arxiv.org/pdf/1704.00028.pdf) introduced an gradient penalty based WGAN and uses $$f$$ as discriminator that achieved state of the arts. 

The training process of WGAN is here:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/WGAN_training.png?raw=true" width="600">
</p>

And the results comparing with other GANs is here:

<p align="center">
<img src="https://github.com/simonzhai/WGAN_Intro/blob/master/images/WGAN_results.png?raw=true" width="600">
</p>

## Acknowledgement
Thanks for [Prof. Hanwang Zhang](http://www.ntu.edu.sg/home/hanwangzhang/) for meaningful suggestions and dedication on this blog.
