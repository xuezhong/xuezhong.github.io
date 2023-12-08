---
author:
- Xuezhong Qiu and Mingyan Liu
bibliography:
- reference.bib
date: 2023-12-07
title: "ST793 Blog: Differential Privacy"
---

In the digital age, our data is constantly at risk of exposure. In this
blog post, we'll explore the concepts of differential privacy and how it
works, and examine the potential privacy attacks it aims to counter.

# Start From k-anonymity

In 1997, a Massachusetts health insurance agency released a database of
hospital visits by state employees and hoped to open it up to
researchers to promote innovation and scientific discovery. For privacy
considerations, they removed columns containing personal information,
including names, phone numbers, full addresses, social security numbers,
etc. What remained was demographic information like zip codes, birth
dates, and gender.

Latanya Sweeney heard about this data release event. Despite the
governor's claim that all identifiers had been removed to protect state
employees' privacy, she realized the action was still not useful enough.
Since the governor himself was a state employee, Sweeney decided to do
the obvious thing and re-identify which records were the governor's.
Sweeney spent some money to purchase public voter records, which
contained full identifiers (name, address) and demographic data (zip
code and birth date), as well as the governor's information. Ultimately,
Sweeney found only one record in the hospital database that matched the
governor's gender, zip code, and birth date. Thus, she was able to learn
information about the governor's prescriptions and hospital visits.
Clearly, the anonymization process was not robust enough.

How to solve this problem? One approach is to ensure the demographic
values in the dataset are no longer unique. A dataset is said to be
k-anonymous if each combination of values for the demographic columns
(such as zip code and age) occurs in at least k different records. We
can simply add mask to some of the demographic values to satisfy
k-anonymity. For example, the following dataset is 2-anonymous:

::: {#tab:my_label}
   ZIP code   age   diagnostic
  ---------- ----- ------------
     42\*     3\*   Common flu
     42\*     3\*    Healthy
     1742     7\*     Ottis
     1742     7\*     Ottis

  : example of 2-anonymity
:::

The intuition is that when a dataset is k-anonymous for a sufficiently
large k, the requirement for a successful re-identification attack is
broken. An attacker might find out the demographic information of their
target using a secondary database, but then this demographic information
will be linked to k different individuals, so it will be impossible to
know which one is their info.

## Is k-anonymity enough?

Now assume an attacker wants to find out Bob's diagnosis. The attacker
knows Bob's zip code is 1742 and her age is over 70. The attacker can
easily deduce that Bob's record is either the third or fourth one.
Though the attacker cannot know specifically which one, he knows Bob's
diagnosis is Ottis. Some other methods can address this problem, such as
l-diversity and t-closeness. All the previous problem definitions
require making some assumptions about the attacker (To pick the right
notion, you need to figure out the attacker's capabilities and goals.
How much background knowledge do they have? What auxiliary data are they
allowed to use? What kind of information are they trying to learn?)

# Differential Privacy: A Paradigm Shift

In 2006, four researchers proposed the concept of differential privacy.
This new approach to privacy protection takes a novel approach to
defining privacy leakage that has proven to be more rigorous and
effective. So what makes differential privacy so special? Why have
governments and tech companies started adopting it for releasing their
data?

### Differential privacy is a more rigorous and general definition. {#differential-privacy-is-a-more-rigorous-and-general-definition. .unnumbered}

Differential privacy protects any kind of information about an
individual. It doesn't matter what the attacker wants to do. Reidentify
their target, know if they're in the dataset, and deduce some sensitive
attribute. All those things are protected. Thus, you don't have to think
about the goals of your attacker. It works no matter what the attacker
knows about your data. They might already know some people in the
database. They might even add some fake users to your system. With
differential privacy, it doesn't matter. The users that the attacker
doesn't know are still protected.

### Differential privacy can quantify the privacy loss {#differential-privacy-can-quantify-the-privacy-loss .unnumbered}

With k-anonymity, there is no clear connection between the value of k
and the level of privacy of the dataset. So choosing k is very
subjective and cannot be formally reasoned about. The issues are even
more severe for the traditional definitions. Differential privacy is
much better in this regard. When using differential privacy, you can
quantify the maximum information gain an attacker could obtain. The
corresponding parameter, called $\epsilon$, allows you to make formal
statements. Suppose $\epsilon$=1.1, then you can say: "An attacker who
thinks their target has a 50% chance of being in the dataset can improve
their confidence to at most 75%." Choosing the exact value of $\epsilon$
is not easy, but at least you can explain it formally.

### Differential privacy can be composed across multiple mechanisms {#differential-privacy-can-be-composed-across-multiple-mechanisms .unnumbered}

Suppose you have some data you want to share anonymously with Alice and
Bob. You trust them equally, so you use the same privacy definition for
both. They are interested in different aspects of the data, so you give
them two different versions of the data. Both versions are \"anonymous\"
according to the definition you chose. But what happens if Alice and Bob
decide to collude and compare the data you gave them? It turns out that
for most privacy definitions, a combination of two anonymous versions is
not anonymous. If you combine two k-anonymous versions of the same data,
the result will not be k-anonymous. So by collaborating, Alice and Bob
may be able to reidentify users themselves or even reconstruct all the
original raw data. This is not good news. However, with differential
privacy, you can avoid this failure mode. Suppose you gave Alice and Bob
differentially private data, each time using parameter $\epsilon$. Then
if they collude, the data they get is still protected by differential
privacy. Now the privacy level is weaker: the parameter becomes
2$\epsilon$. So they still gained some information, but now you can
quantify how much. This property is called composability.

# How Does Differential Privacy Work?

The basic idea is that an algorithm satisfies differential privacy if
its output remains \"basically the same\" after changing the data of one
individual. We can use probability to describe the \"basically the
same\".

## Define differential privacy {#define-differential-privacy .unnumbered}

From [@desfontainesblog20180816], an algorithm A is
$\epsilon$-differentially private if for any database D1 and D2 (which
differ in only one individual) and all possible outcome O :

$$\Pr[A(D_1) = O] \leq e^\varepsilon \cdot \Pr[A(D_2) = O]$$

$\Pr[A(D_1) = O]$ is the probability of getting output O after running
process A on the database D1. The process is probabilistic. For example,
a process might be: \"Count the people with blue eyes, add some random
number to this count, and return this sum\". The results are random
because the random number is different every time the process is
conducted.

$e^\varepsilon$ is an exponential function with the parameter
$\epsilon > 0$. If $\epsilon$ is close to 0, then $e^\varepsilon$ will
be close to 1, thus the probabilities will be \"similar\". The bigger
$\epsilon$ is, the more the probabilities can differ.

The definition is symmetric: the two databases will still differ in only
one individual. So we could replace it by:

$$e^{-\varepsilon} \cdot \Pr[A(D_2) = O] \leq \Pr[A(D_1) = O] \leq e^\varepsilon \cdot \Pr[A(D_2) = O]$$

The formula above means that the output of the process is similar if you
change or remove the data of one person. The degree of similarity is
described with $\epsilon$: a bigger $\epsilon$ stands for a larger
output difference.

What does this similarity have to do with privacy? We'll explain this
with an intuitive example and also formalize this idea with a more
generic interpretation.

### A simple example: randomized response {#a-simple-example-randomized-response .unnumbered}

Consider the following scenario. You need to survey to investigate the
number of people who are illegal drug users. The first action we may
consider is going out and directly asking individuals if they are
illegally using drugs or not. However, many of them might tell a lie.
Therefore we may consider the following method. Each of the participants
will flip a coin, and the participant will not show it to you.

If the coin shows the head, the participant tells the truth (Yes or No).

If the coin shows a tail, the participant flips a second coin. If the
second coin shows a head, the answer would be Yes. If the second coin
shows a tail, the answer would be No.

You might want to know how it works better for survey participants. This
is because participants now know that even if their answers are Yes, it
does not indicate they are using illegal drugs. For participants who say
Yes, they might be illegal drug users, and also might just give a random
answer.

For an illegal drug user, with a probability of 50%, they will tell the
truth (Yes) and with another probability of 50%, they will answer at
random. For the random case, they have a 50% chance to answer Yes, thus
in total a 25% chance to say Yes and 25% chance to say No.

All in all, we get a 75% chance to answer Yes and a 25% chance to answer
No. For someone who is not doing drugs, the probabilities are reversed:
25% chance to answer Yes and 75% to answer No. Using the same notations
from the definition:

-   $P[A(Yes)=Yes]=P[answer \  Yes |\  use \  illegally \ drug]=0.75$

-   $P[A(Yes)=No]=P[answer\  No|\   use \  illegally \ drug]=0.25$

-   $P[A(No)=Yes]=P[answer\  Yes |\  not\ use \  illegally \ drug]=0.25$

-   $P[A(Yes)=No]=P[answer\  No|\  not\ use \  illegally \ drug]=0.75$

Compute the ratio of the probabilities, $\frac{0.75}{0.25}=3$. This
means if we choose a $\epsilon$ such that $e^\epsilon=3$, this process
satisfies the differentially private with degree $\epsilon$.

You're getting some noise into data with a differentially private
process like this. However, if you have enough data, the noise is likely
to cancel out. Assume you collect 10000 answers in total: 4000 of them
are Yes and 6000 are No. As about 2500 answers of Yes and No are random,
you can minus each count by 2500. As a result, you get 1500 Yes answers
out of 5000 non-random answers, and illegal drug users are about 30%.

If you want more privacy, the probability of participants telling the
truth can be set as 25%. If you want less noise, the probability of
participants telling the truth can be set as 75%.

# What Is the New Idea?

Differential privacy can be understood in terms of the following attack:
an attacker gets access to (a randomized version of) algorithm A and its
output y, and tries to infer whether the input database was D1 or D2.
This is often called a \"differential attack\" since the attacker's goal
is to distinguish between two cases.

### How would the attacker achieve a differential attack?  {#how-would-the-attacker-achieve-a-differential-attack .unnumbered}

Differential privacy provides worst-case guarantees, so we analyze the
capabilities of the most powerful attacker. The most powerful attack
algorithm is equivalent to the most powerful statistical hypothesis test
that tries to distinguish between two hypotheses about the input data.
This is known as the null hypothesis and the alternative hypothesis in
the terminology of hypothesis testing. So the most powerful algorithm to
identify the input is equivalent to the \"most powerful\" hypothesis
testing to distinguish between the null hypothesis and alternative
hypothesis as follows:

H0 : y is from database D1 versus H1 : y is from database D2

As such, rejecting the null hypothesis corresponds to the detection of
the absence of y, whereas accepting the null hypothesis means detecting
the presence of y in the database. By framing the privacy analysis in
terms of hypothesis testing, we can leverage that statistical framework
to reason about and quantify privacy guarantees.

The power of a test is the probability that the test correctly rejects
the null hypothesis (H0) when the alternative hypothesis (H1) is true.
This means the most powerful test is the one that has the greatest
ability to detect an effect when there is one while controlling for the
probability of a Type I error (incorrectly rejecting a true null
hypothesis).

The fundamental difficulty in distinguishing the two hypotheses is best
described by the optimal trade-off between the achievable type I and
type II errors. We can consider the two databases D1 and D2 have
distributions of P and Q, respectively. A fundamental constraint in
hypothesis testing from [@dong2022gaussian] is:
$$\alpha_{\phi} + \beta_{\phi} \geq 1 - TV(P,Q)$$

where $\phi$ is the reject rule of hypotheses testing, $\alpha_{\phi}$
is the type I error, $\beta_{\phi}$ is the type II error, $TV(P, Q)$ is
the total variation distance of two probability distributions P and Q.

This inequality characterizes the relationship between the type I error
rate $\alpha_\phi$ (the probability of erroneously rejecting a true null
hypothesis) and the type II error rate $\beta_\phi$ (the probability of
failing to reject an incorrect null hypothesis) for given probability
distributions P and Q.

The total variation distance $TV(P, Q)$ is a measure of the statistical
distance between two probability distributions P and Q. It is defined as
the supremum over all measurable sets A of the absolute difference
between the probabilities assigned to A by P and Q:

$$TV(P,Q) = \sup_{A} |P(A) - Q(A)|$$ Not only is there a constant lower
bound on $\alpha_\phi + \beta_\phi$, but the trade-off between
$\alpha_\phi$ and $\beta_\phi$ also varies as the rejection rule $\phi$
changes. This implies that we should quantify the trade-off between
$\alpha_\phi$ and $\beta_\phi$.

**Definition of trade-off function**: For any two probability
distributions $P$ and $Q$ on the same space, define the trade-off
function $T(P, Q)$ from the interval $[0,1]$ to itself as

$$T(P,Q)(\alpha) = \inf \{\beta_{\phi} : \alpha_{\phi} \leq \alpha \},$$

where the infimum is taken over all (measurable) rejection rules. Then a
question comes, how can we get this function?

According to the **Neyman-Pearson lemma**, for simple null and simple
alternative hypotheses, the most powerful test for a given significance
level $\alpha$ is the likelihood ratio test. The likelihood ratio test
compares the likelihood of the observed data under the null hypothesis
to the likelihood under the alternative hypothesis. This tells us that
when considering the trade-off function $T(P, Q)(\alpha)$, we should use
the likelihood ratio test as the most powerful test. In some simple
cases, we can obtain explicit expressions for the trade-off function. In
the following sections, we will discuss the trade-off functions for
$\epsilon$-DP and Gaussian DP. But if we go further now, the existence
of $T(P, Q)(\alpha)$ will lead us to a new definition of differential
privacy.

**Definition (f-differential privacy)**: Letting f be a trade-off
function, an algorithm A is said to be f-differentially private if

$$T(A(D_1), A(D_2)) \geq f$$ for all neighboring datasets D1 and D2.
Here, neighboring datasets mean they only have one different data row.

This new definition of privacy is parameterized by functions, rather
than real-valued parameters like $\varepsilon$, which offers a more
complete characterization of \"privacy\" and avoids the pitfalls of
summarizing statistical information prematurely.

### Is this f-DP definition consistent with $\epsilon-DP$ ? {#is-this-f-dp-definition-consistent-with-epsilon-dp .unnumbered}

We show that f-DP is a generalization of $\epsilon-DP$. This foreshadows
a deeper connection between f-DP and $\epsilon-DP$. Denote

$$f_{\varepsilon}(\alpha) = \max \{0, 1 - e^{\varepsilon}\alpha, e^{-\varepsilon}(1 - \alpha) \}$$

for $0 \leq \alpha \leq 1$, which is a trade-off function of
$\epsilon-DP$, showed in Figure 1.

![trade-off function $f_{\varepsilon}$ of
$\epsilon-DP$](image/image3.png){#fig:enter-label
width="0.5\\linewidth"}

As showing in Figure 1, trade off function between $\alpha$ and $\beta$
lays on the line $y=max(1-e^{\epsilon}x,e^{-\epsilon}(1-x))$. We will
give a simple explanation to show it is true when $\alpha < \beta$.
Considering a simple implementation of $\epsilon$-DP, it is to add
Laplace noise to the database statistic x to get a random output M(x),
which is
$$M(x)= x + Y, s.t. Pr(Y=y)=\frac{1}{2\epsilon}e^{-\frac{|y|}{\epsilon}}$$
Here we assume X has a sensitivity (dx) of 1 (the max change of X due to
change of target data item is 1) without losing generality. So we can
get the P and Q distribution as following

![an example of P,Q when
$\epsilon$=1](image/image2.png){#fig:enter-label width="0.5\\linewidth"}

The CDF of Laplace noise is $$F(y)=\begin{cases}
    \frac{1}{2}e^{\frac{y}{\epsilon}}, & \text{ if }y < 0\\
    1 - \frac{1}{2}e^{\frac{-y}{\epsilon}}, & \text{ if }y \ge 0\\
\end{cases}$$ Suppose t is the threshold from the likelihood ratio test
with type I error $\alpha$, we can get $$\begin{aligned}
    \alpha = F(-t) = \frac{1}{2}e^{\frac{-t}{\epsilon}}, t > 0 \\
    \beta = 1-F(-(t-1)) = 1 - \frac{1}{2}e^{\frac{-(t-1)}{\epsilon}} = 1-\alpha e^\epsilon
\end{aligned}$$ This example shows the trade-off between type I error
and type II error in the case of $\epsilon-DP$.

As seen in Figure 1, when P and Q are identical to each other, the
trade-off function will be y=1-x, which means the mechanism will ignore
the input and always give random results with the same distribution. In
this situation, the attacker can do nothing except a random guess. Thus,
the nearer the trade-off function is to axes, the more privacy is lost.
The trade-off function gives a tight definition for differential
privacy.

When P and Q are normal distributions, with a difference of $\mu$
between them. We can define the new f-DP as Gaussian Differential
Privacy.

### Gaussian Differential Privacy {#gaussian-differential-privacy .unnumbered}

This subsection introduces a parametric family of f-DP guarantees, where
f is the trade-off function of two normal distributions. We refer to
this specialization as Gaussian differential privacy (GDP). GDP enjoys
many desirable properties. We can now precisely define the trade-off
function with a single parameter. To define this notion, let

$$G_{\mu} := T(N(0,1),N(\mu,1))$$

for $\mu \geq 0$. An explicit expression for the trade-off function
$G_{\mu}$ reads

$$G_{\mu}(\alpha) = \Phi(\Phi^{-1}(1 - \alpha) - \mu),$$

where $\Phi$ denotes the standard normal CDF. This trade-off function is
decreasing in $\mu$ in the sense that $G_{\mu} \leq G_{\mu'}$ if
$\mu \geq \mu'$. We give proof of the equation as follows.

When $\mu \geq 0$, likelihood ratio of $\mathcal{N}(\mu,1)$ and
$\mathcal{N}(0,1)$ is
$\frac{\phi(x-\mu)}{\phi(x)} = e^{\mu x - \frac{1}{2}\mu^2}$, a monotone
increasing function in x. So the likelihood ratio tests must be
thresholds: reject if the sample is greater than some $t$ and accept
otherwise. Assuming $X \sim \mathcal{N}(0,1)$, the corresponding type I
and type II errors are

$$\alpha(t) = \Pr[X > t] = 1 - \Phi(t), \quad \beta(t) = \Pr[X + \mu < t] = \Phi(t - \mu).$$

Solving $\alpha$ from $t$, $t = \Phi^{-1}(1 - \alpha)$. So

$$G_{\mu}(\alpha) = \beta(\alpha) = \Phi(\Phi^{-1}(1 - \alpha) - \mu).$$

We introduce Gaussian Differential Privacy as a specialized case of
differential privacy where the trade-off function is defined between two
normal distributions, making it parameterized by a single parameter
$\mu$. Figure 3 gives the trade-off function plot with different $\mu$.

![Gaussian Differential Privacy](image/image4.png){#fig:enter-label
width="0.5\\linewidth"}

### f-DP definition is a more tight definition regarding composition {#f-dp-definition-is-a-more-tight-definition-regarding-composition .unnumbered}

From $\epsilon-DP$ to $f-DP$, it looks like the definition becomes more
complex. What is the advantage of the f-DP definition? Composition is
one of the most important issues when applying differential privacy in
practice. It is very common that a database has all kinds of queries
related to different statistics of the data, like mean, median, or max.
So the guarantee of privacy with the composition cross different
mechanisms is a key to designing differential privacy mechanisms. $f-DP$
privacy definition is closed and tight under composition, which means
that the trade-off between type I and type II errors that result from
the composition of a $f1-DP$ mechanism with a $f2-DP$ mechanism can
always be exactly described by a certain function f. This function can
be expressed via f1 and f2 in an algebraic fashion, thereby allowing for
lossless reasoning about composition. In contrast, $\epsilon-DP$ or any
other privacy definition artificially restricts itself to a small number
of parameters and results in informal but complicated composability
analysis.

Because of the restriction of time, we can't explore more properties of
f-DP and GDP, but it already shows us many interesting aspects of the
new definition. If you have more interest in continuing, please refer to
more materials listed at the end of the blog.

# Conclusion

This Blog gives an introduction to the history of Differential Privacy.
Compared to k-Anonymity, the $\epsilon-DP$ is a more tight definition,
while the $f-DP$ tries to extend the definition to a higher dimension.
The $f-DP$ has many good properties to be explored. I look forward to
more fruitful research and applications of it.
