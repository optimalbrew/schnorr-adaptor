# Schnorr Adaptor Signature Example

Asked Gemini for sample code in go (it offerred python first). Saved the code in `src`

```
go mod init adaptor_signature_example
```


```
go get github.com/ethereum/go-ethereum
```

then try

```
go run src/adaptor_signature.go
```

## Step by step breakdown from Gemini
Here's a breakdown of the first four steps in constructing Schnorr adaptor signatures:

**1. Foundational Concepts: Schnorr Signatures**

   * **Key Generation:**
        * Choose a secure elliptic curve and a generator point $G$ of a subgroup of prime order $q$.
        * Generate a private key $x$ (a random integer modulo $q$).
        * Compute the public key $P = xG$.
   * **Signing a Message** $m$**:**
        * Choose a random nonce $k$ (an integer modulo $q$).
        * Compute the nonce point $R = kG$.
        * Compute the challenge $e = H(R || P || m)$, where $H$ is a cryptographic hash function.
        * Compute the signature $s = k - ex \pmod{q}$.
        * The signature is the pair $(R, s)$.
   * **Verification of a Signature** $(R, s)$ **on Message** $m$**:**
        * Compute the challenge $e = H(R || P || m)$.
        * Verify if $sG + eP = R$.

**2. Introducing the Adaptor Secret**

   * The core idea of adaptor signatures is to introduce a secret value $t$ into the signing process. This secret allows someone who knows it to "adapt" the initial signature into a valid signature on a different message or under a different condition.

**3. Constructing the Adaptor Signature**

   * Let's say Alice wants to create an adaptor signature on a message $m_1$, such that Bob, upon learning a secret $t$, can derive a valid Schnorr signature on a message $m_2$.
   * **Alice's Setup:**
        * Alice has her private key $x_A$ and public key $P_A = x_A G$.
        * Alice wants to sign $m_1$.
        * Alice chooses a random nonce $k$ and computes $R = kG$.
        * Instead of directly computing the signature $s$, Alice introduces the adaptor secret $t$. She computes a modified "signature" $s' = k - e x_A - t \pmod{q}$, where $e = H(R || P_A || m_1)$.
        * The adaptor signature is $(R, s')$.
   * **Verification of the Adaptor Signature** $(R, s')$ **on** $m_1$ **(without knowing** $t$**):**
        * Compute the challenge $e = H(R || P_A || m_1)$.
        * Verify if $s'G + eP_A = R - tG$. Notice that since $t$ is unknown, a verifier cannot directly confirm this equation. The adaptor signature itself isn't a valid standard Schnorr signature on $m_1$.

**4. Adapting the Signature**

   * Now, suppose Alice reveals the secret $t$ to Bob.
   * **Bob's Adaptation:**
        * Bob receives the adaptor signature $(R, s')$ and the secret $t$.
        * Bob can compute a standard Schnorr signature $(R, s_{adapted})$ on a message $m_2$ (which could be related to the condition under which $t$ was revealed).
        * The adapted signature component $s_{adapted}$ is calculated as: $s_{adapted} = s' + t = (k - e x_A - t) + t = k - e x_A \pmod{q}$.
   * **Verification of the Adapted Signature** $(R, s_{adapted})$ **on** $m_1$ **(using the challenge from** $m_1$**):**
        * Bob (or anyone) can now verify if $(R, s_{adapted})$ is a valid Schnorr signature on $m_1$ using Alice's public key $P_A$:
            * Compute $e = H(R || P_A || m_1)$.
            * Verify if $s_{adapted}G + eP_A = (k - ex_A)G + e(x_A G) = kG - ex_A G + ex_A G = kG = R$.

