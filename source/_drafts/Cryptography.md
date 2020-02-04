---
title: Cryptography
categories:
- Cryptography
tags:
- Cryptography
- Lecture Notes
description: Notes taken from ...
mathjax: true
---

# Mathematical Background

## Discrete Probability

<https://en.wikibooks.org/wiki/High_School_Mathematics_Extensions/Discrete_Probability>

### Information Theoretic Security (Shannon 1949)

> Basic idea: a ciphertext should reveal no information about the plaintext.

But what does infomation about plaintext actually mean?

> ==Definition==： A cipher $(\boldsymbol{E}, \boldsymbol{D})$ over $(\mathbb{K}, \mathbb{M}, \mathbb{C})$ has **perfect secrecy** if
> $$
> \forall m_0, m_1 \in \mathbb{M}, \quad
> \text{len}(m_0)= \text{len}(m_1) \\
> \text{and} \quad \forall c \in \mathbb{C}, \quad \text{Pr}[\boldsymbol{E}(k, m_0)=c] = \text{Pr}[\boldsymbol{E}(k, m_1)=c] \\
> $$
> where $k$ is uniform in $\mathbb{K}$  ($k\stackrel{R}{\longleftarrow}\mathbb{K}$).

Given a ciphertext, I can't tell if the message is $m_0$ or $m_1$ (for all $m_0$ or $m_1$).
The most powerful adversary learns nothong about plaintext form ciphertext.
So ciphertext-only attacks are impossible, but other attacks may be possible.

> ==Theorem==: $\text{perfect secrecy} \Rightarrow |\mathbb{K}| \geq |\mathbb{M}|$.

This is a *bad news* because it means that a cipher with perfect secrecy is not particularly useful. If two parties have a way to agree on really long keys that are as long as the message, then in some sence, they might as well use that mechanism to already transmit the message itself.

# 古典密码

## 单表密码

单表密码只使用一张密码字母表，且明文字母与密文字母有固定的对应关系。

频率分析法可以对付单表密码。

- 加法密码（凯撒密码）
- 乘法密码，定义$a$模$b$的乘法逆元$x$为 $13x \equiv b$
- 仿射密码

## 多表密码

同一明文字母对应多个密文字母

- Playfair
- **Vigenere**
- Beaufort
- Vernam
- Hill
- **Enigma**
  - 加密解密过程见文档
  - double stepping现象：
    如1号齿轮旋转，从Q到R时，会带动第二个齿轮转一次。（相当于从Q到R时进位）



# Symmetric Ciphers

>  ==**Definition**==: a **cipher** defined over $(\mathbb{K}, \mathbb{M}, \mathbb{C})$ is a pair of "efficient" algorithm $(\boldsymbol{E}, \boldsymbol{D})$, where
>  $$
>  \boldsymbol{E}: \mathbb{K} \times \mathbb{M} \to \mathbb{C},\qquad
>  \boldsymbol{D}: \mathbb{K} \times \mathbb{C} \to \mathbb{M} \\
>  s.t. \forall m \in \mathbb{M}, k \in \mathbb{K}: \boldsymbol{D}(k, \boldsymbol{E}(k, m)) = m
>  $$
>  The condition $\forall m \in \mathbb{M}, k \in \mathbb{K}: \boldsymbol{D}(k, \boldsymbol{E}(k, m)) = m$ is called the **correctness property** or **consistency equation**.

The word "efficient" has different meanings in different situations.

The encryption algorithm $\boldsymbol{E}$ is usually randomized, while the deryption algorithm $\boldsymbol{D}$ is always deterministic.

## The One Time Pad (Vernam 1917)

> $$
> \mathbb{M} = \mathbb{C} = \{ 0, 1 \} ^ n, \quad
> \mathbb{K} = \{ 0, 1 \} ^ n \\
> \text{key } k = \text{a random bit string as long as the message} \\
> \boldsymbol{E}(k, m) = k \oplus m, \quad
> \boldsymbol{D}(k, c) = k \oplus c
> $$

It's called One Time Pad because **a key can only be used to encrypt a single message**.

This is a very fast enc/dec algorithm! But it requires long key (as long as plaintext), which is quite problematic.

> ==Lemma==: **OTP has perfect secrecy.**
>
> Proof:
> $$
> \forall m, c: \underset{k}{\text{Pr}}[\boldsymbol{E}(k, m) = c] = \frac{\# \text{keys}:k\in\mathbb{K},s.t.\boldsymbol{E}(k, m) = c}{|\mathbb{K}|} \\
> \therefore \forall m, c: |\{k\in\mathbb{K}:\boldsymbol{E}(k,m）=c\}| = const. \Rightarrow \text{Cipher has perfect secrecy} \\
> \text{For OTP}: \boldsymbol{E}(k, m) = c \Rightarrow k = m \oplus c \\
> \Rightarrow |\{k\in\mathbb{K}:E(k, m)=c\}| = 1, \quad \forall m, c \\
> \Rightarrow \text{OTP has perfect secrecy.}
> $$

So OTP has no ciphertext-only attacks.

But the required key length is too large and makes it impratical. But due to a former theorem proved by Shannon, this is already the minimal length that keeps **perfect secrecy**.

## Stream Ciphers: making OTP practical

> idea: replace "random" key by "pseudorandom" key with a **pseudorandom generetor**.
> $$
> \text{PRG:} \quad \boldsymbol{G}: \{0, 1\}^s \text{(seed space)} \to \{0, 1\}^n, \quad n \gg s
> $$
> where $\boldsymbol{G}$ is a "effciently" computable deterministic algorithm.
> $$
> \mathbb{M} = \mathbb{C} = \{ 0, 1 \} ^ n, \quad
> \mathbb{K} = \{ 0, 1 \} ^ s, \quad s < n \\
> \boldsymbol{E}(k, m) = \boldsymbol{G}(k) \oplus m, \quad
> \boldsymbol{D}(k, c) = \boldsymbol{G}(k) \oplus c
> $$

Stream ciphers cannot have perfect secrecy! Therefore, 

- we need a different definition of security
- security will depend on specific PRG

### PRG must be unpredictable

> ==Definition==: PRG $\boldsymbol{G}: \mathbb{K} \to \{0, 1\}^n$ is **predictable** if
> $$
> \exists \text{"efficient" algorithm } \boldsymbol{A} \text{ and } 1 \leq i \leq n-1 \\
> s.t. \underset{k\overset{R}{\longleftarrow}\mathbb{K}}{\text{Pr}} [\boldsymbol{A}(\boldsymbol{G}(k)|_{1,...,i})=\boldsymbol{G}(k)|_{i+1}] > \frac{1}{2} + \epsilon \\
> \text{for some "non-negligible" } \epsilon \text{, such as } \epsilon > \frac{1}{2^{30}}
> $$
> ==Definition==: PRG is **unpredictable** if it is not predictable.
> $$
> \forall i: \text{no "efficient" adversary can predict bit } i+1 \text{ for "non-negligible" } \epsilon
> $$

Suppose PRG is predictable, $ \exists i: \boldsymbol{G}(k) |_{1, ..., i} \overset{alg.}{\longrightarrow} \boldsymbol{G} |_{i+1, ..., n}$, the stream cipher would not be secure.

If an attacker acutally intercepts a particular ciphertext $c$, and by some prior knowledge, the attacker actually knows the value of initial part of the message, then it could xor the ciphertext with the known prefix of the message gives a prefix of the pseudorandom sequence, with which the attacker can predict the remainder of the pseudorandom sequence and thus the remainder of the message.

### Weak PRGs

#### Linear congruential generator

> ==Defnition==: 
> $$
> \text{Linear congruential generator with params } a, b, p \\
> r[0] \equiv \text{seed}, \quad r[i] \gets a \cdot r[i-1] + b \text{ mod } p \text{ for } i > 0
> $$

Easy to predict despite of some good statistical properties!

#### glibc's random()

> ==Definition==:
> $$
> \begin{align}
> & r[i] \gets (r[i-3] + r[i-31]) \text{ mod } 2^{32} \\
> & \text{output } r[i] >> 1
> \end{align}
> $$

Never use glibc's built-in `random()` function for crypto!

### Negligible and non-negligible

**In practice**, $\epsilon$ is a scalar and 

- $\epsilon$ is non-neg: $\epsilon \geq \frac{1}{2^{30}}$ (likely to happen over 1GB of data)
- $\epsilon$ is negligible: $\epsilon \leq \frac{1}{2^{80}}$ (won't happenn over life of a key)

**In theory**, $\epsilon$ is a function $\epsilon: \mathbb{Z}^{\geq 0} \to \mathbb{R}^{\geq 0}$ and

- $\epsilon$ is non-neg: $\exists d: \epsilon(\lambda) \geq \frac{1}{\lambda^d}$ inf. often ($\epsilon \geq \frac{1}{\text{polynomial}}$, for many $\lambda$)
- $\epsilon$ is negligible: $\forall d, \lambda \geq \lambda_d: \epsilon(\lambda) \leq \frac{1}{\lambda^d}$ ( $\epsilon \leq \frac{1}{\text{polynomial}} $, for large $\lambda$)

## Attacks on Stream Ciphers and OTP

### Attack 1: two time pad is insecure

>  Never use stream cipher key more than once!

$$
c_1 \gets m_1 \oplus \boldsymbol{\text{PRG}}(k) \\
c_2 \gets m_2 \oplus \boldsymbol{\text{PRG}}(k)
$$

Eavesdropper does:
$$
c_1 \oplus c_2 \to m_1 \oplus m_2
$$
Enough redundancy in English and ASCII encoding that:
$$
m_1 \oplus m_2 \to m_1, m_2
$$

- Network traffic: negotiate new key for every session (e.g. TLS)
- Disk encryption: typically do not use a stream cipher

### Attack 2:  no integrity (OTP is malleable)

if an acitve attacker modifies the ciphertext $m \oplus k$ to $m \oplus k \oplus p$, the decryption result would be $m \oplus p$.

Modifications to cipheretxts are **undetected** and have **predictable impact** on plaintext.

## Real-World Stream Ciphers

### Old example (software): RC4 (1987)

> ==Descryption==:
>
> 1. use a 128-bit seed
> 2. generate a 2048-bit code as its internal state
> 3. does an infinate loop: each iteration outputs a byte

RC4 is used in HTTPs and WEP.

#### Weaknesses

- Bias in initial output: $\text{Pr}[2^{\text{nd}} \text{ byte} = 0] = \frac{2}{256},$ and acutually the first and third bytes are also slightly biased
  - in practice, should ignore the first 256 bytes of the RC4 output and start using from 257th byte
- Another bias: $\text{Pr}[(0, 0)]=\frac{1}{256^2}+\frac{1}{256^3}$
- Related key attacks

### Old example (hardware): CSS (badly broken)

#### Linear feedback shift register (LFSR)

A linear-feedback shift register (LFSR) is a shift register whose input bit is a linear function of its previous state. And the most commonly used linear function is XOR.

The initial values of LFSR is called the seed.

LFSR is used in (following designs are all **badly broken**):

- DVD encryption (CSS): 2 LFSRs
- GSM encryption (A5/1.2): 3LFSRs
- Bluetooth (E0): 4 LFSRs

> ==Descryption==: **CSS**
>
> 1. use 5 bytes = 40 bits as the key
> 2. use a 17-bit LFSR and a 25-bit LFSR
> 3. Concatenate $1$ to the first two bytes of the key, and make it the seed of the 17-bit LFSR
> 4. Concatenate $1$ to the last three bytes of the key, and make it the seed of the 25-bit LFSR
> 5. Run the LFSRs for 8 cycles and genereate 8 bits of output.
> 6. Go through an adder that does addition modulo 256, with the carry bit of the previous block
> 7. So the adder gives one byte per round

#### How to break CSS

CSS is easy to break within approximately $2^{17}$ time.

Since DVD is using MPEG files, if you know a prefix of the plaintext. And if you XOR the plaintext prefix with the ciphertext, you'll get the first a few bytes of the output of the PRG. Let's say the length of the prefix is $a$ bytes.

Now, try all $2^{17}$ possible values of the first 17-bit LFSR. For each value as its initial value, run the LFSR for $a$ bytes. 

Next, we take the $a$ bytes of the ciphertext (output of the target CSS system) and subtract it from the $a$ bytes from the first LFSR. Note that if out guess for the initial state of the first LFSR is correct, we should get the first $a$-byte output of the second LFSR. And it's easy to tell whether this $a$-byte sequence really came from the second LFSR or not. And if it didn't, move on to the next candidate for the 17-bit LFSR until we hit the right one.

And then, we'll learn the correct initial states of both LFSR. Therefore the remaining outputs of CSS can be predicted and the ciphertext can be decrypted.

> The reason the key of CSS is limited to 40 bits is that DVD encryption was designed at a time when U.S. Export regulations only allowed for exports of crypto algorithms where the key was only 40 bits.

### Modern stream ciphers: eStream (2008)

> $$
> \text{PRG: } \{0, 1\}^s \times R \to \{0, 1\}^n, \quad n \gg s
> $$
>
> where $\{0, 1\}^s$ denotes a seed and $R$ denotes **nonce**.
>
> ==Definition==: **Nonce** is a non-repeating value for a given key.
> $$
> E(k, m; r) = m \oplus \text{PRG} (k; r)
> $$
> The pair $(k, r)$ is never used more than once. The bottom line is that the key can be re-used because the nonce makes the pair unique.

#### eStream: Salsa 20 (SW+HW)

> ==Definition==: **Salsa 20**
> $$
> \text{Salsa 20: }\{0, 1\}^{128\text{ or }256} \times \{0, 1\}^{64} \to \{0, 1\}^n \quad
> \text{(max } n = 2^{73} \text{ bits)}
> $$
> where the $\{0, 1\}^{128\text{ or }256}$ is the seed and the $\{0, 1\}^{64}$ is the nonce.
>
> The function is defined as follows:
> $$
> \text{Salsa 20} (k; r) = H(k, (r, 0)) || H(k, (r, 1)) || \dots
> $$
> The function $H$ takes three inputs, the seed $k$, the nonce $r$, and a counter that increments step by step. And the function $H$ starts off by expanding the states into a 64-byte block, which should look as the table below shows (128-bit version of Salsa).
>
> |    Content    | length(bytes) |
> | :-----------: | :-----------: |
> |  Constant 0   |       4       |
> |      Key      |      16       |
> |  Constant 1   |       4       |
> |     Nonce     |       8       |
> | Counter/Index |       8       |
> |  Constant 2   |       4       |
> |      Key      |      16       |
> |  Constant 3   |       4       |
>
> Then apply an invertible function $h$, which maps 64 bytes to 64 bytes, to the above 64-byte block. Repeat it ten times in total. Note that since $h$ is invertible, it should be easy to invert the output at this point to its initial state.
>
> So there is one more thing to be done, which is to basically do an 4-byte addition module $2^{32}$ word by word between the initial state and the output. And this gives the output of function $H$.
>
> >  Here the function $h$ is designed specifically so that:
> >  - it's easy to implement in hardware
> >  - x86 has SSE2 instruction set which supports all the operations needed for this function, it runs fast on x86.

## What is a secure cipher?

### PRG Security Definitions

Consider a PRG with keyspace $\mathbb{K}$ that outputs $n$-bit strings. Out **goal** is to define that 
$$
[k \overset{R}{\longleftarrow} \mathbb{K}, \quad \text{output } G(k)]
$$
is **"indistinguishable"** from 
$$
[r \overset{R}{\longleftarrow}\{0, 1\}^n, \quad \text{output } r]
$$

#### Statistical Tests

To understand how to define this concept of **indistinguishability from random**, we need the concept of a **statistical test**.

> ==Definition==: **Statistical test on $\{0, 1\}^n$**
> $$
> \text{an algorithm }\boldsymbol{A} \\
> s.t. \boldsymbol{A}(x) \text{ outputs "0" or "1"}
> $$
> Basically a statistical test is an algorithm that takes its input $x$, an $n$-bit string, and simply outpus $0$ or $1$, which represents that the input given is not random or random respectively.

##### Advantage

How do we evaluate a stastical test is good or not?

> Let $\boldsymbol{G:}\mathbb{K} \to \{0, 1\}^n$ be a PRG and $\boldsymbol{A}$ be a statistacal test on $\{0, 1\}^n$.
>
> ==Definition==: The **advantage** $\text{Adv}_{\text{PRG}}$ of the statistical test $\boldsymbol{A}$ relative to this generator $\boldsymbol{G}$ is defined as follows:
> $$
> \text{Adv}_{\text{PRG}} [\boldsymbol{A}, \boldsymbol{G}] =
> \left| \underset{k \overset{R}{\longleftarrow} \mathbb{K}}{\text{Pr}} [\boldsymbol{A}(\boldsymbol{G}(k)) = 1] - \underset{r \overset{R}{\longleftarrow} \{0, 1\}^n}{\text{Pr}} [\boldsymbol{A}(r) = 1] \right|
> \in [0, 1]
> $$
> 

- if $\text{Adv}$ is close to $1$, it means $\boldsymbol{A}$ *can* distinguish $\boldsymbol{G}$ from random.
- if $\text{Adv}$ is close to $0$, it means $\boldsymbol{A}$ *cannot* distinguish $\boldsymbol{G}$ from random.

#### Secure PRGs: crypto definition

> ==Definition==:
>
> We say that $\boldsymbol{G}:\mathbb{K} \to \{0, 1\}^n$ is a **secure PRG** if 
> $$
> \forall \text{ "efficient" statistical tests } \boldsymbol{A}: \\
> \text{Adv}_{\text{PRG}} [\boldsymbol{A}, \boldsymbol{G}] \text{ is "negligible".}
> $$

The restriction to the **"efficient"** statistical tests is actually necessary. If we ask that all statistical tests, regardless of whether they're efficient or not, not be able to distinguisha the output from random, then in fact, <u>that can not be satisfied</u>.

But are there **provably secure PRGs**? The answer is, it's unknown. The reason is that if you could prove that a particular generator is secure, that would actually imply that $P \neq NP$. And in fact if $P = NP$, it's easy to show that there are no secure PRGs.

#### Easy fact: a secure PRG is unpredictable

[In a previous segment](#PRG must be unpredictable), we discussed that a PRG is **unpredictable** basically means that given a prefix of the output generator, it's impossible to predict the next bit of the output.

Now we can show that PRG is predictable $\Rightarrow$ PRG is insecure.

> Suppose $\boldsymbol{A}$ is an efficient algorithm, $s.t. \underset{k \overset{R}{\longleftarrow} \mathbb{K}}{\text{Pr}} \left[\boldsymbol{A}(\boldsymbol{G}(k)|_{1,\dots\,i}) = G(k)|_{i+1}\right] = \dfrac{1}{2} + \epsilon$ 
> for non-negligible $\epsilon$ (e.g. $\epsilon = \dfrac{1}{1000}$).
>
> Define statistical test $\boldsymbol{B}$ as:
> $$
> \boldsymbol{B}(x) = 
> \begin{cases}
> 1, &\text{if } \boldsymbol{A}(x|_{1,\dots,i}) = x_{i+1} \\
> 0, &\text{else}
> \end{cases}
> $$
> Now we have:
> $$
> \begin{cases}
> r \overset{R}{\longleftarrow} \{0, 1\}^n, &\text{Pr}\left[\boldsymbol{B}(r)=1\right] = \dfrac{1}{2} \\
> k \overset{R}{\longleftarrow} \mathbb{K}, &\text{Pr}\left[\boldsymbol{B}(\boldsymbol{G}(k))=1\right] = \dfrac{1}{2} + \epsilon
> \end{cases} \\
> \Rightarrow \text{Adv}_{\text{PRG}}\left[\boldsymbol{B}, \boldsymbol{G}\right] > \epsilon
> $$
> Therefore, if a predictor algorithm $\boldsymbol{A}$ is able to predict the next bits with advantage $\epsilon$, then a statistical test algorithm $\boldsymbol{B}$ is able to distinguish the output of the generator with advantage $\epsilon$.

#### Theorem: an unpredictable PRG is secure (Yao, 1982)

It shows that the converse is also true.

> Let $\boldsymbol{G}:\mathbb{K} \to \{0, 1\}^n$ be PRG. 
>
> ==Theorem==: If $\forall i \in \{0, \dots, n-1\}$ PRG $\boldsymbol{G}$ is unpredictable at position $i$, then $\boldsymbol{G}$ is a secure PRG.

In other words, if next-bit predictors cannot distinguish $\boldsymbol{G}$ from random, then no statistical test can.

#### More generally

> Let $P_1$ and $P_2$ be two distrubutions over $\{0, 1\}^n$
>
> ==Definition==: $P_1$ and $P_2$ are **computationally indistuinguishable** (denoted $P_1 \approx_P P_2$), if
> $$
> \forall \text{ "efficient" statistical tests } \boldsymbol{A} \\
> \left| \underset{x \gets P_1}{\text{Pr}} \left[\boldsymbol{A}(x)=1\right] - \underset{x \gets P_2}{\text{Pr}} \left[\boldsymbol{A}(x)=1\right] \right|
> < \epsilon_0
> $$
> where $\epsilon_0$ is a negligible value.

So PRG $\boldsymbol{G}$ is secure if $\left\{\boldsymbol{G}: k \overset{R}{\longleftarrow} \mathbb{K} \right\} \approx_P \text{Uniform}(\{0, 1\}^n)$.

### Semantic Security

What does it mean for a cipher to be secure? 

Recal Shannon's [Perfect Secrecy](# Information Theoretic Security (Shannon 1949)) (ciphertexts should reveal **no information** about the plaintexts). And we know that this definition is too strong because it requires the keys to be too long, and therefore stream ciphers cannot satisfy this difinition.

So a weakened version of this difinition is that we can require the two distributions just to be **computationally indistinguishable**. But it turns out that this definition is still a little too strong and cannot be satisfied. So one more constraint is required that this definition needs to hold for only $m_0, m_1$ pairs that the attackers could actually exhibit.

And this actually leads to the definition of semantic security.

> ==Definition==: **Semantic Security (one-time key)**
>
> For $b=0, 1$, define experiment $\text{EXP}(b)$ as:
>
> We have an attacker and a challenger. The attacker output two messages $m_0$ and $m_1$, which is an explicit pair of messages that the attacker wants to be challenged on and are of equal length. 
> And then the challenger outputs either the encryption of $m_0$ if $b=0$, or that of $m_1$ if $b=1$.
> Finally the attacker tries to guess the value of $b$ and outputs $\text{EXP}(b) \in \{0, 1\}$ as the answer. 
>
> Define events $W_b= \text{event that EXP(b)=1}$ for $b = 0, 1$.
>
> Define the advantage of the adversary $\text{Adv}_{SS}[A,E] = \left| \text{Pr}[W_0] - \text{Pr}[W_1] \right| \in [0, 1]$, where $A$ denotes the adversary and $E$ denotes the encryption scheme.
> So if $\text{Adv}_{SS}[A, E]$ is close to $0$, it means the adversary $A$ is not able to distinguish $\text{EXP}(0)$ from $\text{EXP}(1)$; and if $\text{Adv}_{SS}[A, E]$ is close to $1$, it means otherwise.
>
> Finally we define that $E$ is **semantically secure** if for all "efficient" $A$, $\text{Adv}_{SS}[A, E]$ is "negligible".

From this definition we can learn that if $E$ is semantically secure, for all explicit $m_0, m_1 \in \mathbb{M}$: $\left\{E(k,m_0)\right\} \approx_p \left\{E(k,m_1)\right\} $.

#### Examples

![](1554881210515.png)

![](1554881433071.png)

#### Stream Ciphers are Semantically Secure [TODO]

> ==Theorem==:
> $$
> G:K \to \{0, 1\}^n \text{ is a secure PRG} \\
> \Rightarrow \text{stream cipher } E \text{ derived from } G \text{ is semantically secure.} \\
> \forall \text{semantically secure adversary } A, \exists \text{ a PRG adversary } B \\
> s.t. \text{Adv}_{SS}[A, E] \leq 2 \times \text{Adv}_{\text{PRG}}[B, G]
> $$

## Block Ciphers

A block cipher is made up of two algorithms, $E$ and $D$, which are encryption and decryption algorithms and both take a key $k$ as input. The point of a block cipher is that it takes an $n$-bit plaintext as input and outputs exactly $n$ bits of outputs.

Canonical examples:

- 3DES (triple-DES): $n=64 \text{ bits}$, $k=168\text{ bits}$.
- AES: $n=128 \text{bits}$, $k= 128, 192, 256 \text{ bits}$

Block ciphers are usually built by iterations.

![](1554882563905.png)

Here $R(k_i, m)$ is called a round function.

For 3DES, $n=48$; for AES, $n=10$.

### Abstractly: PRPs and PRFs

> ==Definition==: **Pseudo Random Function (PRF)** defined over $(K, X, Y)$:
> $$
> F: K \times X \to Y
> $$
> such that "efficient" algorithm to evaluate $F(k, x)$.

> ==Definition==: **Pseudo Random Permutation (PRP)** defined over $(K, X)$:
> $$
> E: K \times X \to X
> $$
> such that:
>
> - exists "efficient" <u>determinstic</u> algorithm to evaluate $E(k, x)$
> - the function $E(k, \cdot)$ is one-to-one, i.e. bijective
> - exists "efficient" inversion algorithm $D(k, y)$ 

Example PRPs: 3DES, AES, ...

- AES: $K \times X \to X$ where $K = X = \{0, 1\}^{128}$
- 3DES: $K \times X \to X$ where $X=\{0, 1\}^{64}, K=\{0, 1\}^{168}$

Functionally, any PRP is also a PRF. Actually, a PRP is a PRF where $X=Y$ and is "efficiently" invertible.

#### Secure PRFs

> Let $F: K \times X \to Y$ be a PRF
> $$
> \left\{
> \begin{array}
> \text{Funs} [X,Y]: \text{ the set of all functions from } X \text{ to } Y \\
> S_F = \{ F(k, \cdot) \quad s.t. \, k \in K \} \subseteq \text{Funs}[X,Y]
> \end{array}
> \right.
> $$
> ==Intuition==: a PRF is **secure** if
>
> - a random function in $\text{Funs}[X, Y]$ is indistinguishable from a random function in $S_F$.

Note that the size $\left| \text{Funs}[X,Y] \right|$ is equal to size $\left| Y \right|^{\left| X \right|}$, while the size $\left| S_F \right|$ is equal to size $|K|$, which is obviously much much smaller.

Consider an adversary who is trying to distinguish a truly random function from a pseudo-random function. By repeatedly submitting messages to his target function, he always receives either the output of a truly random function or that of a pseudo-random one. Now the idea is clear that if the PRF is secure, this poor adversary will not be able to tell the difference.

One thing should be noted is that even if you have a secure PRF,  it's enough that on just one known input the output is kind of not random, the output is fixed; and already the PRF is broken, even though you realize that everywhere else the PRF is perfectly indistinguishable from random. 

By the way, the definition of a **secure PRP** is pretty much the same, except that $F$ should be limited to permutations of $X$, i.e. one-to-one functions.

#### An easy application: PRF => PRG

> Let $F: K \times \{0, 1\}^n \to \{0, 1\}^n$ be a secure PRF.
>
> Then the following $G: K \to \{0, 1\}^{nt}$ is a secure PRG:
> $$
> G(k) = F(k,0) || F(k, 1) || \dots || F(k, t)
> $$
> where $||$ denotes concatenation.

Here the key property is being **parallelizable**. In other words, blocks to be concatenated can be computed by different cores. And by the way, most stream ciphers, such as RC4, are strictly sequential.

#### PRF Switching Lemma

Any secure PRP is also a secure PRF, if $|\mathbb{X}|$ is sufficiently large.

> ==Lemma==: PRF Switching Lemma
>
> Let $E$ be a PRP over $(\mathbb{K}, \mathbb{X})$. Then for any $q$-query adversary $\mathcal{A}$:
> $$
> \left|
> \text{Adv}_{\text{PRF}}[\mathcal{A}, E] - \text{Adv}_{\text{PRP}}[\mathcal{A}, E]
> \right|
> < \dfrac{q^2}{2|\mathbb{X}|}
> $$

Suppose $|\mathbb{X}|$ is large so that $\dfrac{q^2}{2|\mathbb{X}|}$ is "neglibible", then
$$
\text{Adv}_{\text{PRP}}[\mathcal{A}, E] \text{ is "neglibible" } \Rightarrow \text{Adv}_{\text{PRF}}[\mathcal{A}, E] \text{ is "neglibible" }
$$

### Data Encryption Standard (DES)

#### History

- Early 1970s:
  - Horst Feistel designs **Lucifer** at IBM
  - key-len = 128 bits
  - block-len = 128 bits
- 1973:
  - NBS asks for block cipher proposals
  - IBM submits variant of **Lucifer**
- 1976:
  - NBS adopts **DES** as a federal standard
  - key-len = 56 bits
  - block-len = 64 bits
- 1997:
  - DES broken by exhaustive search
- 2000:
  - NIST adopts Rijndael as **AES** to replace DES

#### core idea - Feistel Network

Given functions $f_1, \dots, f_d: \{0, 1\}^n\to \{0, 1\}^n $

Goal: build invertible function $F: \{0, 1\}^{2n} \to \{0, 1\}^{2n}$

![](feistel_network.png)

> ==Claim==: for all $f_1, \dots, f_d: \{0, 1\}^n \to \{0, 1\}^n$, Feistel network $F: \{0, 1\}^{2n} \to \{0, 1\}^{2n}$ is invertible.
>
> Proof: construct inverse
>
> ![](feistel_network_inverse.png)

- Inversion is basically the same circuit, with $f_1, \dots, f_d$ applied in a reverse order
- Feistel Network is a general method for building invertible functions (block ciphers) from arbitrary functions.
- Used in many block ciphers... but not AES!

> ==Theorem==: (Luby-Rackoff '85) <a name='luby-rackoff'></a>
> $$
> f: K \times \{0, 1\}^n \to \{0, 1\}^n \text{ is a secure PRF.} \\
> \Rightarrow \text{3-round Feistel } F: K^3 \times \{0, 1\}^{2n} \to \{0, 1\}^{2n} \text{ is a secure PRP.}
> $$
>

Since a true definition of a secure block cipher is that it needs to be a secure pseudo-random permutation, this theorem tells us that if you start with a secure pseudo-random function, you end up with a secure block cipher.

#### DES: 16-round Feistel network

ref: 

- <https://en.wikipedia.org/wiki/Data_Encryption_Standard>
- <https://en.wikipedia.org/wiki/DES_supplementary_material>

#### Choosing the S-box and P-box

Chossing the S-boxes and P-boxes **at random** would result in an insecure block cipher (key recovery after $\approx 2^{24}$ outputs) [BS' 89]. It turns out that random choice would tend to be somewhat close to linear functions.

And so the designers of DES specified a number of rules they use for choosing the S-boxes:

- No output bit should be close to a linear function of the input bits
- S-boxes are 4-to-1 maps
- ...

#### Exhaustive Search Attacks agianst DES

> ==Goal==: given a few input output pairs $\left(m_i , c_i = E(k, m_i)\right), i=1, \dots, 3$, find key $k$.

> ==Lemma==: 
>
> Suppose DES is an **ideal cipher** ($2^{56}$ random invertible functions $\Pi_1, \dots, \Pi_{2^{56}}: \{0, 1\}^{64} \to \{0, 1\}$).
>
> Then $\forall m, c$, there is at most **one** key $k$, $s.t. \, c = \text{DES}(k, m) $, with a probability $\geq 1 - \dfrac{1}{256} \approx 99.5\%$.
>
> ***Proof***:
> $$
> \begin{align}
> \begin{split}
> &      & \text{Pr}\left[\exists k' \neq k: c=\text{DES}(k, m)=\text{DES}(k', m)\right] \\
> & \leq & \sum_{k' \in \{0, 1\}^{56}} \text{Pr} \left[ \text{DES}(k, m) = \text{DES}(k', m) \right] \\
> & \leq & \quad \dfrac{1}{2^{64}} \times 2^{56} \\
> & =    & \quad \dfrac{1}{2^8}
> \end{split}
> \end{align}
> $$

For two DES pairs $(m_1, c_1=\text{DES}(k, m_1)), (m_2, c_2=\text{DES}(k, m_2))$, the unicity probability $\approx 1-\dfrac{1}{2^{71}}$.

For AES-128, given two input/output pairs, the unicity probability $\approx 1-\dfrac{1}{2^{128}}$.

Therefore, two input/output pairs are enough for exhaustive key search.

#### Strengthening DES against exhaustive search

##### Method 1: Triple-DES

- Let $E: K \times M \to M$ be a block cipher.
  Define $3E: K^3 \times M \to M$ as $3E((k_1, k_2, k_3), m) = E(k_1, D(k_2, E(k_3, m)))$.
- key size = $3 \times 56 = 168$ bits
- three times slower than DES
- there is a simple attack in time $\approx 2^{118}$

> But actually, anything that's larger than $2^{90}$ can be considered sufficiently secure.

But what's wrong with **double-DES**?

Let's define $2E((k_1, k_2), m) = E(k_1, E(k_2, m))$ (with a key length of 112 bits for DES).

<u>Meet-in-the-middle Attack</u>: 
$$
\begin{align}
\begin{split}
& \, \text{find} (k_1, k_2) \quad s.t. E(k_1, E(k_2, m)) = c \\
\Leftrightarrow & \, \text{find} (k_1, k_2) \quad s.t. E(k_2, m) = D(k_1, c)
\end{split}
\end{align}
$$

1. build a table, where the $i$-th entry is the encryption of the message with the $i$-th key $k^i$.
   The computation and sorting of the table takes $2^{56} \cdot log(2^{56})$ time.
2. for all $k \in \{0, 1\}^{56}$, test if $D(k, c)$ is in the table. If so, then $E(k^i, m) = D(k, c) \Rightarrow (k^i, k) = (k_2, k_1)$.

The *time* required is $2^{56}log(2^{56}) + 2^{56}log(2^{56}) < 2^{63} \ll 2^{112}$.

The *space* required is $\approx 2^{56}$. (But nevermind, so be it.)

>  The same attack applied on triple-DES: *time* $= 2^{118}$, *space* $\approx 2^{56}$.

##### Method 2: DESX

- Let $E: K \times \{0, 1\}^n \to \{0, 1\}$ be a blcok cipher.
  Define $EX((k_1, k_2, k_3), m) = k_1 \oplus E(k_2, m \oplus k_3)$.
- key size $= 64+56+64 = 184$ bits
- but there is an easy attack in time $2^{64+56} = 2^{120}$

Note that $k_1 \oplus E(k_2, m)$ and $E(k_2, m \oplus k_1)$ actually does nothing! It'll still be as vulnerable as the original block cipher $E$.

>  Unlike triple-DES, DESX is not standardized by NIST, because it doesn't defend against more subtle attacks on DES.

### More Attacks on Block Ciphers

#### Attacks on the implementation

##### Side channel attacks

Measure *time/power* to do encryption/decryption.

There's an attack called **differential power analysis**, where you measure the power consumed by the card over many rounds of the encryption algorithm. And as long as there's any even small dependence between the amount of current consumed and the bits of the secret key, basically that dependence will show up after enough rounds of the encryption algorithm. And as a result you'll be able to completely extract the secret key.

As far as **timing attacks** are concerned, for example, imagine a multicore processor where the encryption algorithm is running on one core while the attacker's code is running on another. Now these cores actually share the same cache. And as a result, an attacker can actually measure or can look at the exact cache missis that the encryption algorithm incurred, and completely figure out the secret key used by the algorithms.

##### Fault attacks

If you are attacking a smart-card/processor, you can cause it to malfunction and output erroneous data. And it turns out that if errors occured during the last round of the encryption process, the resulting ciphertexts that are produced are enough to actually expose the secret key.

So the defense against this means that before you output the result of your algorithm, you should check to make sure that the correct result was computed. Now of course that's nontrivial, because how do you know that the error didn't happen in your checking algorithm? But there are known ways around that.

So basically you can actually compute something three or four times, take the majority over all those results, and be assured that the output really is correct as long as not too many faults occurred inside of your computation.

> Do not even **implement** crypot primitives yourself!

#### Linear and differential attacks

So the goal of an attack is, given many input/output pairs, can we recover the key faster than exhaustive search?

##### Linear cryptanalysis

> overview: Let $c = E(k, m)$, suppose for random $k, m$:
> $$
> \text{Pr} \big[m[i_1] \oplus \dots \oplus m[i_r] \oplus c[j_j] \oplus \dots \oplus c[j_v] = k[l_1] \oplus \dots \oplus k[l_u] \big]
> = \dfrac{1}{2} + \epsilon
> $$
> for some $\epsilon$.

For $E=\text{DES}$, this exists with $\epsilon = \dfrac{1}{2^{21}} \approx 0.0000000477$. It so happens because of some bugs of the fifth S-box by design. It turns out the fifth S-box happens to be too close to a linear function, which propagates through the entire DES circuit, and generates a relation of this type.

> ==Theorem==: Given $\dfrac{1}{\epsilon^2}$ random $(m, c=\text{DES}(k, m))$ pairs, then
> $$
> k[l_1, \dots, l_u] = \text{MAJ} \big[
> m[i_1, \dots, i_r] \oplus c[j_j, \dots, j_v]
> \big]
> $$
> with probability $ \geq 97.7\%$.

With $\dfrac{1}{\epsilon^2}$ input/output pairs, we can find $k[l_1, \dots, l_u]$ in time $\approx \dfrac{1}{\epsilon^2}$.

For DES, $\epsilon = \dfrac{1}{2^{21}}$. So with $2^{42}$ input/output pairs, we can find $k[l_1, \dots, l_u]$ in time $2^{42}$. Roughly speaking, we can find $14$ key bits this way in time $2^{42}$. Now just brute force the remaining $56-14=42$ bits in time $2^{42}$. So the total attack time $\approx 2^{43} \ll 2^{56}$ with $2^{42}$ random input/output pairs.

> A tiny bit of linearity in $S_5$ leads to a $2^{42}$ time attack.
>
> So do not design ciphers yourself!

#### Quantum attacks [TODO]

> **Generic search problem**:
>
> Let $f: \mathbb{X} \to \{0, 1\}$ be a function.
>
> Goal: find $x \in \mathbb{X}, \, s.t. f(x)=1$.

With a classical computer, the best generic algorithm for generic search problem has time complexity $O(|\mathbb{X}|)$.

But with a quantum computer [Grover '96], this problem can be solved in time $O(|\mathbb{X}|^{\frac{1}{2}})$.

- [ ] extra material can be found in discussion forum

> **Quantum exhaustive search**:
>
> Given $m, c=E(k, m)$, define
> $$
> f(k) = 
> \left\{
> \begin{array}\\
> 1, & \text{if } E(k, m) = c \\
> 0, & \text{otherwise}
> \end{array}
> \right.
> $$
> And by Grover's algorithm, a quantum computer can find $k$ in time $O(|\mathbb{K}|^{\frac{1}{2}})$

- DES: time $\approx 2^{28}$
- AES-128: time $\approx 2^{64}$

### Advanced Encryption Standard (AES)

#### History

- 1997: NIST publishes request for proposal
- 1998: 15 submissions & 5 claimed attacks
- 1999: NIST chooses 5 finalists
- 2000: NIST chooses Rijndael as AES (designed in Belgium)
  - key sizes: 128, 192, 256 bits
  - block size: 128 bits

#### Substitution-Permutation Network

![](subs-perm-network.png)

And the inversion of a substitution-permutation network is simply done by applying all of the steps in the reverse order. So each step should be made reversible by design, which different from Feistel Network.

#### AES-128 schematic

Check [the flash animation](Rijndael_Animation_v4_eng.swf).

#### code-size/performance tradeoff

![](aes-tradeoff.png)

#### AES in hardware

AES instructions in Intel Westmere:

- **aesenc**, **aesenclast**:
  - do one round of AES
  - 128-bit registers: `xmm1`=state, `xmm2`=round-key
  - `aesenc xmm1, xmm2 ;puts result in xmm1`
- **aeskeygenassist**:
  - performs AES key expansion
- claim 14 x speed-up over OpenSSL on same hardware

There are similar instructions on AMD Bulldozer and future architectures.

#### Attacks on AES

##### best key recovery attack

Only four times better than exhaustive search [BKR '11].

##### Related key attack on AES-256 [BK '09]

Given $2^{99}$ input/output pairs from **four related keys** (with very short Hamming distance) in AES-256, keys can be recovered in time $\approx 2^{99}$.

### Block Ciphers from PRGs

#### Build a PRF from a PRG

> ==Theorem==:
> Let $G: \mathbb{K}\to \mathbb{K}^2$ be a secure PRG. Define 1-bit PRF $F: \mathbb{K} \times \{0, 1\} \to \mathbb{K}$ as 
> $$
> F (k, x \in \{0, 1\}) = G(k)[x]
> $$
> If $G$ is a secure PRG, then $F$ is a secure PRF.

But can we build a PRF with a larger domain?

Let $G: \mathbb{K} \to \mathbb{K}^2$ be a secure PRG. Define $G_1: \mathbb{K} \to \mathbb{K}^4$ as $G_1(k) = G\big(G(k)[0]\big) || G\big(G(k)[1]\big)$.
We get a 2-bit PRF: $F_1(k, x \in \{0, 1\}^2) = G_1(k)[x]$.

It's easy to prove that $G_1$ is also a secure PRG and therefore $F_1$ is a secure PRF with proof by contradiction.

#### Extending: the GGM PRF

Let $G: \mathbb{K} \to \mathbb{K}^2$ be a secure PRG. Define PRF $F: \mathbb{K} \times \{0, 1\}^n \to \mathbb{K}$ as
$$
k_i = G(k_{i-1})[x_{i-1}], \quad 1 \leq i \leq n,\quad k_0=k \\
F(k, x = x_0 x_1 \dots x_{n-1} \in \{0, 1\}^n) = k_n
$$
If $G$ is a secure PRG, then $F$ is a secure PRF on $\{0, 1\}^n$.

But this GGM PRF is not used in practice dut to its slow performance.

#### Secure block cipher (PRP) from a PRG

As we said before, if we have a secure PRF, we can plug it into the [Luby-Rackoff theorem](#luby-rackoff) and get a provably secure PRP. 

But again this is considerably slower than heuristic constructions like AES.

### Modes of operation: one-time key

#### insecure: Electronic Code Book (ECB)

- Break the message into blocks of equal length (16 bytes in the case of AES).
- Encrypt each block separately.

This is terribly **insecure** because if two blocks are equal, then the resulting ciphertext is also going to be equal. And as a result, the attacker learns something about the plaintext that he should't have learned.

And ECB is also not [semantically secure](#Semantic Security) for messages that contain more than one block. And the advantage can be as high as $1$.

#### secure: Deterministic Counter

- Suppose we have a PRF $F$ (e.g. AES) and we need to encrypt a $L$-block long message $m$ with key $k$
- for $0 \leq i \leq L-1$, $c[i] = m[i] \oplus F(k, i)$

> ==Theorem==: Deterministic Counter mode security
>
> For any $L > 0$, if $F$ is a seucre PRF over $(\mathbb{K, X, X} )$, then $E_{\text{DETCTR}}$ is a semantically secure cipher over $(\mathbb{K}, \mathbb{X}^L, \mathbb{X}^L)$.
>
> In particular, for any "efficient" adversary $\mathcal{A}$ attacking $E_{\text{DETCTR}}$, there exists an "efficient" PRF adversary $\mathcal{B}$, such that
> $$
> \text{Adv}_{SS}[\mathcal{A}, E_{\text{DETCTR}}] = 2 \cdot \text{Adv}_{\text{PRF}}[\mathcal{B}, F]
> $$
> $\text{Adv}_{\text{PRF}}[\mathcal{B}, F]$ is negligible (since $F$ is a secure PRF). Hence, $\text{Adv}_{SS}[\mathcal{A}, E_{\text{DETCTR}}]$ must be negligible. 
>
> ![](detctr_sec_proof.png)

### Modes of operation: many-time key

example applications:

- File systems: the same AES key used to encrypt many files
- IPsec: the same AES key used to encrypt many packets

#### Semantic Security for many-time key (CPA security)

When we use the key more than once, the result is that the adversary gets to see many ciphertexts encrypted using the same key. As a result, when we define security, we're gonna allow the adversary to mount what's called a **chosen-plaintext attack**.

- **Adversary's power**: chosen-plaintext attack (CPA)
  - can obtain the encryption of arbitrary messages of his choice (conservative modeling of real life)
- **Adversary's goal**: break semantic security

> ==Definition==: Semantic Security under CPA
>
> $\mathcal{E} = (E, D)$ is a cipher defined over $(\mathbb{K, M, C})$.
>
> For $b=0, 1$, define experiment $\text{EXP}(b)$ as:
>
> ![](sem-sec-CPA.png)
>
> $\mathcal{E}$ is semantic secure under CPA if for all "efficient" adversary $\mathcal{A}$:
>
> $\text{Adv}_{\text{CPA}}[\mathcal{A}, \mathcal{E}] = \Big| \text{Pr}[\text{EXP}(0)=1] - \text{Pr}[\text{EXP}[1]=1] \Big|$ is "negligible".

##### Ciphers insecure under CPA

If an encryption scheme $E$ always outputs the same ciphertext for a particular message $m$, then $E$ is insecure under CPA (e.g. one time pad and deterministic counter mode).

###### Solution 1: randomized encryption

$E(k, m)$ is a randomized algorithm which has a high probability of giving different ciphertexts when encrypting the same message twice.

In this case, the ciphertexts must be longer than the plaintexts because the randomness used to generate the ciphertexts will be encoded somehow in the ciphertexts.

Roughly Speaking, $\text{size}_{\text{CT}} = \text{size}_{\text{PT}} + \#\text{"random bits"}$.

###### Solution 2: nonce-based encryption

![](nonce-based-enc.png)

- nonce $n$: a value that changes from message to message.
  - $(k, n)$ pair is never used more than once.
  - Nonce is not required to be random. It only needs to be unique each time.

**method 1**: nonce is a **counter **(e.g. packet counter)

- used when ecryptor keeps state from message to message
- if decryptor has the same state, need not send noce with ciphertexts

**method 2**: encryptor choooses a **random** nonce, $n \gets \mathbb{N}$.

- $|\mathbb{N}|$ should be large enough so that the nonce will never repeat for the life of the key with a high probability.
- encryption state is not required (thus no need to coordinate between encryptors and decryptors)

> When we talk about nonce-based encryption, we generally allow the adversary to choose the nonce, and the system should remain secure.
>
> ==Definition==: CPA security for nonce-based encryption
>
> ![](nonce-based-CPA.png)
>
> Note the restriction that all nonces $\{n_1, \dots, n_q\}$ that the adversary chooses must be distinct. And as a result, the adversary will never see messages ecrypted using the same nonce.
>
> Now we define that a nonce-based encryption system $\mathcal{E}$ is semantically secure under CPA if for all "efficient" adversary $\mathcal{A}$: $\text{Adv}_{\text{nCPA}}[\mathcal{A}, \mathcal{E}] = \Big | \text{Pr}[\text{EXP}(0)=1] - \text{Pr}[\text{EXP}(1)=1] \Big |$ is "negligible".

#### Cipher block chaining (CBC)

##### Construction 1: CBC with random IV

Let $(E, D)$ be a PRP. $E_{\text{CBC}}(k, m)$ chooses a **random** $\text{IV} \in \mathbb{X}$ that's exactly one block of the block cipher, and do the following:

![](CBC_random_IV.png)

Here IV stands for initialization vector. And so the ciphertext is a little longer than the plaintext because we had to include this IV in the ciphertexts, which basically captures the randomness that was used during encryption.

##### CBC: CPA analysis

> ==CBC theorem==: 
>
> For any $L > 0$, if $E$ is a secure PRP over $(\mathbb{K, X})$, then $E_{\text{CBC}}$ is semantically secure under CPA over $(\mathbb{K}, \mathbb{X}^L, \mathbb{X}^{L+1})$.
>
> In particular, for a $q$-query adversary $\mathcal{A}$ attacking $E_{\text{CBC}}$, there exists a PRP adversary $\mathcal{B}$, such that:
> $$
> \text{Adv}_{CPA}[\mathcal{A}, E_{\text{CPA}}]
> \leq 2 \cdot \text{Adv}_{\text{PRP}}[\mathcal{B}, E] + \dfrac{2 q^2 L^2}{|\mathbb{X}|}
> $$

Here the $\dfrac{2 q^2 L^2}{|\mathbb{X}|}$ is usually referred to as the "error term".

CBC is only secure as long as $q^2 L^2 \ll |\mathbb{X}|$.

Suppose we want $\text{Adv}_{\text{CPA}}[\mathcal{A}, E_{\text{CBC}}] \leq \dfrac{1}{2^{32}}$ because we know $\dfrac{q^2 L^2}{|\mathbb{X}|} < \dfrac{1}{2^{32}}$, we have:

- AES: $|\mathbb{X}|=2^{128} \Rightarrow qL < 2^{48}$
  - So, after $2^{48}$ AES blocks, the key must be changed.
- 3DES: $|\mathbb{X}| = 2^{64} \Rightarrow qL < 2^{16}$

> ==Warning==: CBC where attackers can **predict** the IV is **not** CPA-secure!
>
> Actually there is a bug in SSL/TLS 1.1: IV for record #i​ is the last ciphertext block for record #i-1.

##### Construction 1': nonce-based CBC

In cipher block chaining with <u>unique</u> nonce, the key is actually pair $(k, k_1)$, where $k$ is used multiple times to encrypt the message, and $k_1$ is used to encrypt the nonce.

![](CBC-nonce-based.png)

##### A CBC technicality: padding

So what if the last block of the message is shorter than the block size? The answer is that we add a pad at the end so it becomes as long as the block size.

> A typical pad in TLS:
>
> For $n > 0$, $n$-byte pad is $n||n||\dots||n$. And the decryptor look at the very last byte of the message and remove the pad. If no pad is needed, the encryptor will add a dummy block, where all bytes are 16.

There is also a variant of CBC call [CBC with ciphertext stealing](<https://en.wikipedia.org/wiki/Ciphertext_stealing#CBC_ciphertext_stealing_decryption_using_a_standard_CBC_interface>) that actually avoids this problem.

#### Randomized counter (CTR)

##### Construction 2: randomized counter mode

Unlike CBC, randomized counter mode uses a secure PRF. It doesn't need a block cypher. It's enough for counter mode to just use a PRF because we're never going to be inverting this function.

![](1557409964433.png)

As you noticed, IV is included in the ciphertext, so the ciphertext is a little longer than the original plaintext.

One thing should be noted is that CTR mode is **paralyzable**, unlike CBC which is inherently sequential.

##### Construction 2': nonce-based CTR-mode

![](1557410423098.png)

The only restriction is that you encrypt at most $2^{64}$ blocks using one particular nonce. Otherwise there would at least two blocks encrypted with the same OTP.

##### rand ctr-mode (rand. IV): CPA analysis

> ==Counter-mode Theorem==:
>
> For any $L > 0$, if $F$ is a secure PRF over $\mathbb{(K, X, X)}$, then $E_{\text{CTR}}$ is semantically secure under CPA over $(\mathbb{K}, \mathbb{X}^L, \mathbb{X}^{L+1})$.
>
> In particular, for a $q$-query adversary $\mathcal{A}$ attacking $E_{\text{CTR}}$, there exists a PRF adversary $\mathcal{B}$ s.t.:
> $$
> \text{Adv}_{\text{CPA}}[\mathcal{A}, E_{\text{CTR}}] \leq 2 \cdot \text{Adv}_{\text{PRF}}[\mathcal{B}, F] + \dfrac{2 q^2 L}{|\mathbb{X}|}
> $$
>
> > Note: CTR-mode is only secure when $q^2 L \ll |\mathbb{X}|$. Better than CBC!

#### Comparison: CTR vs. CBC

|                           |           CBC           |          CTR          |
| ------------------------- | :---------------------: | :-------------------: |
| uses                      |           PRP           |          PRF          |
| parallel processing       |           No            |          Yes          |
| Security of rand. enc     | $q^2L^2\ll|\mathbb{X}|$ | $q^2L\ll|\mathbb{X}|$ |
| dummy padding block       |           Yes           |          No           |
| 1 byte msgs (nonce-based) |      16x expansion      |     no expansion      |

First of all, recall that CBC actually had to use a block cypher because if you look at the decryption circuit, the decryption circuit actually ran the block cypher in reverse. It was actually using the decryption capabilities of the block cypher. Whereas in counter mode, we only use a PRF. We never use the decryption capabilities of the block cypher. We only use it in the forward direction, only encrypt with it. Because of this, **counter mode is actually more general** and you can use primitives like Salsa.

The security bounds, the error terms are better for counter mode than they are for CBC. And as a result, you can use a key to encrypt more blocks in counter mode than you could with CBC.

(For CBC, dummy padding block can be solved using ciphertext stealing.)

Finally, suppose you're encrypting just a stream of one byte messages, and using nonce encryption with an implicit nonce. So, the nonce is not included in the cipher text. In this case, every single one byte message would have to be expanded into a sixteen-byte block and then encrypted. And the result would be a sixteen-byte block.  In counter mode, of course, this is not a problem. You would just encrypt each one byte message by XORing with the first bytes of the stream that's generated in the counter mode. So every cipher text would just be one byte, just like the corresponding plain text.

So you see that essentially, in every single aspect counter mode dominates CBC.
And that's why it's actually the recommended mode to be using today.

### Summary

- PRPs and PRFs: a useful abstraction of block ciphers.

- We examined two security notions: (security against eavesdropping):

  1. Semantic security against one-time CPA.
  2. Semantic security against many-time CPA.

  Note: neither mode ensures data integrety.

- Stated security results summarized in the following table:
  ![](1557413062043.png)

# Message Integrity

In this module, we're gonna instead discuss message integrity and show how to provide both encryption and integrity. So our goal here is to provide integrity without any confidentiality.



# Hash Functions

MD5

