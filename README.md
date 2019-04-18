# PHBS_BlockChain_2018

## ZeroCash and sk-SNARK

### 1. Zero knowledge proof
Zero knowledge proof was proposed by Goldwasser in 1989 and was a two-party or multi-party agreement. It allows one party (the prover) to prove to another (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself. Take the currency transaction as an example. It means the transaction can be proved valid without revealing the payer, the payee, or the transaction amount.

In 2013, based on zero-knowledge proof, [Miers](https://ieeexplore.ieee.org/document/6547123) proposed ZeroCoin, which realized a completely anonymous digital currency transaction, but there are still many problems with ZeroCoin. For example, it does not support non-interactive transactions, does not protect the transaction amount and the privacy of the payee, and the currency amount can not be arbitrarily divided. To this end, many new schemes have been proposed. The ZeroCash proposed by [SASSON](https://ieeexplore.ieee.org/document/6956581) is the most typical.

### 2. [How ZeroCash works](https://blog.z.cash/zcash-private-transactions/)
Based on ZeroCoin, ZeroCash adopts zk-SNARK technology, which can effectively protect the information of the payer, payee and the payment amount, and achieves full anonymity. This part is to provide a simplified explanation of how privacy-preserving transactions work in ZeroCash, and where exactly Zero knowledge proof comes into the picture.

In Bitcoin, UTXO is the basic transaction unit, and ZeroCash uses note as the basic transaction unit. Simplify the note as `note = (PK, v, r)`, `PK` is the owner’s public key (address), `v` is the amount, and `r` is the serial number that can uniquely distinguish the note. 

The transactions in ZeroCash have two categories, transparent addresses and hidden addresses. The input and output of the transparent address transaction are directly visible note information. For hidden address transactions, the input and output are no longer plaintext notes, but the note nullifier and note commitment.

* #### Note commitment

It works as an output of the transaction and indicates a new note has been issued. A valid commitment is a proof of a spendable note. However, we need to make sure that the information it contains does not reveal which note it is, who the owner is and how much amount it is. Therefore, we can hash the information of the note. And the commit corresponding to the note can be simply described as `HASH(note)`.

* #### Note nullifier

It works as an input of the transaction and indicates an old note will be void. Like Bitcoin, the input of one transaction must be the output of another transaction. So the nullifier corresponds to a commitment uniquely, but it should avoid disclosing any information about which commitment the nullifier is related to. To construct a nullifier that satisfies the requirements, it is still a good idea to take a hash. Therefore, the nullifier corresponding to the note with the serial number r can be described as `HASH(r)`.

Bitcoin tracks unspent transaction outputs (UTXOs) to determine what transactions are spendable. In ZeroCash, spending a commitment involves revealing a nullifier. ZeroCash nodes keep lists of all the commitments that have been created and all the nullifiers that have been revealed.

Suppose there are three notes currently, `note1=(PK1,v1,r1)`，`note2=(PK2,v2,r2)`，`note3=(PK3,v3,r3)`. The `note1` belongs to Anna, and `note2` has been spent. The contents of The nullifier and commitment list maintained by each node at this time is shown in Table 1.1.

Commitment Set | Nullifier Set
-------------- | -------------
`C1 = HASH(note1)` | `NF1 = HASH(r2)`
`C2 = HASH(note2)` |
`C3 = HASH(note3)` |

Table 1.1 The commitment and nullifier list before payment


Anna decided to transfer the `note1` to Carl. His public key is `PK4`, she will do this:
* Randomly pick a serial number `r4` and use this to generate `note4 = (PK4, v1, r4)`. 
* Secretly send `note4` to Carl.
* Broadcast the nullifier of `note1` (`NF2 = HASH(r1)`) and newly generated commitment of `note4` (`HASH (note4)`) to all nodes.

The node that receives the Anna’s broadcast will check if `NF2` has already existed in the nullifier set. If not, then the corresponding note is valid. The node will add `HASH (note4)` and `NF2` to the commit and nullifier set maintained by itself, as shown in Table 1.2. The role of nullifier is to prevent digital currency from double spending.

Commitment Set | Nullifier Set
-------------- | -------------
`C1 = HASH(note1)` | `NF1 = HASH(r2)`
`C2 = HASH(note2)` | `NF2 = HASH(r1)`
`C3 = HASH(note3)` |
`C4 = HASH(note4)` |

Table 1.2 The commitment and nullifier list after payment


The above is the principle of ZeroCash, but the following problems will occur.
* Make sure that the note corresponding to `NF2` given by Anna does exist.
* Even if `NF2` does point to a note, make sure Anna has the right to use it.

These two issues can of course be proved by publishing the contents of `note1`, but this will reveal Anna's data privacy. The zk-SNARK comes in handy at this time. Anna will also provide an evidence `π`. `π` is sufficient to prove that Anna knows that `PK1`, `sk1` (the private key corresponding to `PK1`) and `r1` which satisfy the following conditions.
* Use `PK1` and `r1` to restore the note. If its hash value exists in the commitment list, it shows  the note used for payment is valid.
* If `sk1` is the private key of `PK1`, Anna has the right to use this note.
* If `HASH(r1) = NF2`, nullifier is consistent with commitment.

Other nodes acknowledge that the transaction is legal after verifying the `π` is valid. At the same time, the properties of zk-SNARK will ensure no information about `PK1`, `sk1` and `r1` can be revealed by `π`. 

### 3. [How zk-SNARK works](https://z.cash/technology/zksnarks)
ZeroCash uses zk-SNARK to prove that the conditions for a valid transaction have been satisfied without revealing any crucial information about the addresses or values involved. The acronym zk-SNARK stands for zero knowledge Succinct Non-interactive Argument of Knowledge.
* #### Zero knowledge

Zero knowledge proof allows the prover to prove to the verifier that a statement is true, without revealing any information beyond the validity of the statement itself.
* #### Succinct

It means that the transaction verification process does not involve a large amount of data transmission and the verification algorithm is simple.
* #### Non-interactive

The prover and the verifier don’t need to exchange information many times during transaction verification. In ZeroCoin, there are many interactions between prover and verifier to achieve verification reliability, and zk-SNARK attempts to completely avoid these interactions.

The zk-SNARK is a comprehensive application of mathematical theory such as algebraic number theory and abstract algebra. The principle is not deduced in detail here. Figure 1.1 shows the mathematical methods used in it to achieve succinctness, anti-counterfeiting, non-interaction and so on.
<img width="648" alt="principle" src="https://github.com/YunxiaShi/PHBS_BlockChain_2018/blob/master/principle.jpg">

Figure 1.1 The principle of zk-SNARK


* #### Describe the problem as a QAP
As a mathematical method, zk-SNARK must have quantifiable input, so we must first establish a mathematical model for the target problem. If the calculation equation corresponding to the target problem holds when the particular input values are substituted into it, then these inputs are called the "solutions" of the problem. What zk-SNARK has to deal with is to prove "I know the solution."

zk-SNARK is only suitable for a specific form of computational problem, the so-called QAP (Quadratic Arithmetic Programs), which writes the problem into a polynomial equation: `t(x) h(x) = w(x) v(x)`. The prover needs to verify to the verifier that he knows the solution to this equation without showing the solution.

* #### Sample to achieve a concise proof
For ZeroCash, the degree of polynomial can be up to 2,000,000 which means that a large amount of data (polynomial millions of coefficient values) needs to be transmitted for each verification, which obviously does not meet the requirements of zk-SNARK simplicity. The verifier randomly selects a sampling point s to simplify the problem of polynomial multiplication and the verification of equality of polynomial functions into a simple multiplication and the verification of the equation `t(s)h(s) = w(s)v(s)`. This not only reduces the size of the proof, but also greatly reduces the time required for verification.

But this method has a problem. If the prover knows the sampling point s, even if he does not know the correct solution of the problem, he can construct `t'(x)`, `h'(x)`, `w'(x)`, `v(x)`. So that at least at the sampling point `s`, `t'(s) h'(s) = w'(s)v(s)`. The following Homomorphic Hiding can solve this problem by not allowing the sampling point s to be known by the prover, but the prover can also give the value of the polynomial at the sampling point.

* #### Use homomorphic mapping to hide sampling points
Homomorphic Hidden is a property of mapping E that satisfies the following conditions.
1.	For most `x`,  we can’t derive `x` from `X=E(x)`.
2.	If `x1≠x2`，then `E(x1)≠E(x2)`.
3.	`E(ax1+bx2) = a * E(x1) + b * E(x2)`，that is, the addition is homomorphic. After mapping, the calculation form of the addition is still preserved.

Instead of directly telling the sampling point to the prover, the verifier provides the mapping values `E(1)`, `E(s^1)`, `E(s^2)`, …, `E(s^N)` of `s^0`, `s^1`, `s^2`,...`s^N`. The polynomials `t(s)`, `h(s)`, `w(s)`, `v(s)` are linear combinations of `{s^n, n=(0,1,2,3...,N)}`. According to the properties of homomorphic mapping, `E(t(s))`, `E(h(s))`, `E(w(s))`, `E(v(s))` should also be the linear combinations `{E(s^n), n=(0,1,2,3...,N)}`. This allows the prover to calculate `E(t(s))`, `E(h(s))`, `E(w(s))`, `E(v(s))` without knowing `s`. But to prove `E(t(s)h(s)) = E(w(s)v(s))`, the homomorphic hiding problem of multiplication needs to be solved.

* #### Bilinear map: homomorphic hiding of multiplication
The homomorphic hiding introduced earlier is one-to-one, mapping an input to an output. A bilinear map maps two elements from two domains to one element in the third field: `e(X, Y) → Z`, and has linearity on both inputs: 

`e(P+R, Q) = e(P, Q) + e(R, Q)`

`e(P, Q+S) = e(P, Q) + e(P, S)`

Assuming that for any two factorizations of  `x`, `(a, b)` and `(c, d)` (ie `x = ab = cd`), there are two additive homomorphic maps `E1` and `E2`, and a bilinear map `e`, such that the equation is always true: `e(E1(a), E2(b)) = e(E1(c), E2(d)) = X`. Then, the mapping of `x->X` is also an additive homomorphic mapping, denoted as `E`, then `E(xy) = e(E1(x), E2(y))`。

`E(t(s)h(s)) = e(E1(t(s)), E2(h(s)))`

`E(w(s)v(s)) = e(E1(w(s)), E2(v(s)))`

`E(t(s)h(s)) = E(w(s)v(s))`

Thus, the homomorphic hiding problem of multiplication can be solved.
