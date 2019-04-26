# PHBS_BlockChain_2018

## ZeroCash and zk-SNARK

### 1. Introduction
Blockchain is a kind of distributed ledger technology with the characteristics of decentralization, transparent and secure. It integrates peer-to-peer transmission, consensus mechanisms and digital encryption technology. Each participating node in the system holds a complete copy of the data. They maintain the integrity of the data and can effectively avoid the risk of single-point crash and data leakage of the central server.

However, the global ledger that records transaction data in the blockchain is public. And because the validity of the transaction is required to retrieve historical transactions during the consensus process, all transaction information cannot be directly protected by encryption. This increases the risk of data privacy breaches. Dynamic network properties of the transaction, such as coin flow and the number of edge outputs and inputs, contribute further to reveal account identity.[7]

In traditional data privacy protection schemes, the data is stored in the central server. The data management center can protect the data privacy by using the encryption technology, such as [K-anonymity](https://www.computer.org/csdl/proceedings-article/icde/2005/22850217/12OmNvTBB6p) and [homomorphic encryption technology](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.371.8983). However, these schemes do not apply to blockchain. Many solutions have been proposed  for the data privacy leakage problem. The main idea is to hide some of the information in the public data without affecting the normal operation of the blockchain system, and to increase the difficulty of data analysis. This paper introduces the mixed currency mechanism and cryptography technology in the existing privacy protection scheme, and introduces the ZeroCash and zk-SNARK technology in detail.

### 2. Data privacy of blockchain
#### 2.1 Anonymity
The most common approach for protecting the privacy of individuals is to anonymize data before releasing it. This approach is based on the assumption that if the data contains no identifying information about individuals (names, addresses, etc.) it should be safe for release. Unfortunately, individuals can often be identified in anonymized datasets using so-called re-identification attacks. In fact, the anonymity of digital currency needs to achieve pseudonym and unlinkability.

Digital currency such as bitcoin, its users participate in transactions using Bitcoin addresses. The transaction address is formed by Hash transformation and Base58 coding, and has a certain degree of pseudonym. However, since the transaction record is public and traceable, the transaction rules of the address, such as the transaction frequency, transaction characteristics, and the relationship between the addresses, can be analyzed according to the records in the global ledger. So Bitcoin does not guarantee [unlinkability of transactions](https://link.springer.com/chapter/10.1007%2F978-3-642-39884-1_2).

[Reid](https://link.springer.com/chapter/10.1007%2F978-1-4614-4139-7_10?LI=true) analyzed the input and output relationships in the trading network by constructing a trading network and a user network, and obtained multiple inputs finally aggregate to an address, indicating that multi-input transactions are generally initiated by the same owner. 

#### 2.2 Deanonymous attack
Trackers can deanonymize users of cryptocurrencies based on address clustering techniques. It is trivial to generate new Bitcoin addresses, and most wallet software takes advantage of this feature to improve user privacy. In the normal course of operation, users end up with coins split between numerous addresses, and it may not be obvious which addresses belong to the same user (or entity). However, address clustering technique can implement clustering of different addresses to obtain multiple addresses of the same user and obtain user privacy.

The following are two clustering heuristics to link addresses controlled by the same user.
* Multi-input transactions
For a transaction with multiple input addresses, each input address needs to be signed separately, so the input address of most multi-input transactions comes from the same user[6].

* Change addresses
The Bitcoin protocol automatically generates a new change address in which the excess from the input address is sent back to the sender. When the set of output addresses contains two addresses, one has not appeared in any previous transaction and all the other output addresses in the transaction have appeared in previous transactions. We can conclude that the former one is a change address and is controlled by the sender[6].

### 3. Data privacy protection technology of blockchain
In order to improve the anonymity of blockchain technology, protect users’ identity privacy and transaction data privacy, a variety of privacy protection schemes have been proposed, including mixed currency mechanism and encryption protocol technology. Coinjoin, CoinSuffle, Mixcoin, Blindcoin are digital currency that use mixed currency mechanism. The existing technology mainly relies on the third-party platform to mix the transaction sets of multiple users and output them to the corresponding addresses, so that the attacker cannot link the input and output addresses of the transaction. The other is to construct a decentralized anonymous "digital currency" system through cryptography techniques such as multi-party secure computing, blind signature, ring signature and zero-knowledge proof, such as Monero, ZeroCoin and ZeroCash. The following is a comprehensive analysis of ZeroCash and zk-SNARK technology.

#### 3.1 Zero knowledge proof
Zero knowledge proof was proposed by Goldwasser in 1989 and was a two-party or multi-party agreement. It allows one party (the prover) to prove to another (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself. Take the currency transaction as an example. It means the transaction can be proved valid without revealing the payer, the payee, or the transaction amount.

In 2013, based on zero-knowledge proof, [Miers](https://ieeexplore.ieee.org/document/6547123) proposed ZeroCoin, which realized a completely anonymous digital currency transaction, but there are still many problems with ZeroCoin. For example, it does not support non-interactive transactions, does not protect the transaction amount and the privacy of the payee, and the currency amount can not be arbitrarily divided. To this end, many new schemes have been proposed. The ZeroCash proposed by [SASSON](https://ieeexplore.ieee.org/document/6956581) is the most typical.

#### 3.2 [How ZeroCash works](https://blog.z.cash/zcash-private-transactions/)
Based on ZeroCoin, ZeroCash adopts zk-SNARK technology, which can effectively protect the information of the payer, payee and the payment amount, and achieves full anonymity. This part is to provide a simplified explanation of how privacy-preserving transactions work in ZeroCash, and where exactly Zero knowledge proof comes into the picture.

In Bitcoin, UTXO is the basic transaction unit, and ZeroCash uses note as the basic transaction unit. Simplify the note as `note = (PK, v, r)`, `PK` is the owner’s public key (address), `v` is the amount, and `r` is the serial number that can uniquely distinguish the note. 

The transactions in ZeroCash have two categories, transparent addresses and hidden addresses. The input and output of the transparent address transaction are directly visible note information. For hidden address transactions, the input and output are no longer plaintext notes, but the note nullifier and note commitment.

* #### Note commitment

It works as an output of the transaction and indicates a new note has been issued. A valid commitment is a proof of a spendable note. However, we need to make sure that the information it contains does not reveal which note it is, who the owner is and how much amount it is. Therefore, we can hash the information of the note. And the commit corresponding to the note can be simply described as `HASH(note)`.

* #### Note nullifier

It works as an input of the transaction and indicates an old note will be void. Like Bitcoin, the input of one transaction must be the output of another transaction. So the nullifier corresponds to a commitment uniquely, but it should avoid disclosing any information about which commitment the nullifier is related to. To construct a nullifier that satisfies the requirements, it is still a good idea to take a hash. Therefore, the nullifier corresponding to the note with the serial number r can be described as `HASH(r)`.

Bitcoin tracks unspent transaction outputs (UTXOs) to determine what transactions are spendable. In ZeroCash, spending a commitment involves revealing a nullifier. ZeroCash nodes keep lists of all the commitments that have been created and all the nullifiers that have been revealed.

Suppose there are three notes currently, `note1=(PK1,v1,r1)`，`note2=(PK2,v2,r2)`，`note3=(PK3,v3,r3)`. The `note1` belongs to Anna, and `note2` has been spent. The contents of The nullifier and commitment list maintained by each node at this time is shown in Table 3.1.

Commitment Set | Nullifier Set
-------------- | -------------
`C1 = HASH(note1)` | `NF1 = HASH(r2)`
`C2 = HASH(note2)` |
`C3 = HASH(note3)` |

Table 3.1 The commitment and nullifier list before payment


Anna decided to transfer the `note1` to Carl. His public key is `PK4`, she will do this:
* Randomly pick a serial number `r4` and use this to generate `note4 = (PK4, v1, r4)`. 
* Secretly send `note4` to Carl.
* Broadcast the nullifier of `note1` (`NF2 = HASH(r1)`) and newly generated commitment of `note4` (`HASH (note4)`) to all nodes.

The node that receives the Anna’s broadcast will check if `NF2` has already existed in the nullifier set. If not, then the corresponding note is valid. The node will add `HASH (note4)` and `NF2` to the commit and nullifier set maintained by itself, as shown in Table 3.2. The role of nullifier is to prevent digital currency from double spending.

Commitment Set | Nullifier Set
-------------- | -------------
`C1 = HASH(note1)` | `NF1 = HASH(r2)`
`C2 = HASH(note2)` | `NF2 = HASH(r1)`
`C3 = HASH(note3)` |
`C4 = HASH(note4)` |

Table 3.2 The commitment and nullifier list after payment


The above is the principle of ZeroCash, but the following problems will occur.
* Make sure that the note corresponding to `NF2` given by Anna does exist.
* Even if `NF2` does point to a note, make sure Anna has the right to use it.

These two issues can of course be proved by publishing the contents of `note1`, but this will reveal Anna's data privacy. The zk-SNARK comes in handy at this time. Anna will also provide an evidence `π`. `π` is sufficient to prove that Anna knows that `PK1`, `sk1` (the private key corresponding to `PK1`) and `r1` which satisfy the following conditions.
* Use `PK1` and `r1` to restore the note. If its hash value exists in the commitment list, it shows  the note used for payment is valid.
* If `sk1` is the private key of `PK1`, Anna has the right to use this note.
* If `HASH(r1) = NF2`, nullifier is consistent with commitment.

Other nodes acknowledge that the transaction is legal after verifying the `π` is valid. At the same time, the properties of zk-SNARK will ensure no information about `PK1`, `sk1` and `r1` can be revealed by `π`. 

#### 3.3 [How zk-SNARK works](https://z.cash/technology/zksnarks)
ZeroCash uses zk-SNARK to prove that the conditions for a valid transaction have been satisfied without revealing any crucial information about the addresses or values involved. The acronym zk-SNARK stands for zero knowledge Succinct Non-interactive Argument of Knowledge.
* #### Zero knowledge

Zero knowledge proof allows the prover to prove to the verifier that a statement is true, without revealing any information beyond the validity of the statement itself.
* #### Succinct

It means that the transaction verification process does not involve a large amount of data transmission and the verification algorithm is simple.
* #### Non-interactive

The prover and the verifier don’t need to exchange information many times during transaction verification. In ZeroCoin, there are many interactions between prover and verifier to achieve verification reliability, and zk-SNARK attempts to completely avoid these interactions.

The zk-SNARK is a comprehensive application of mathematical theory such as algebraic number theory and abstract algebra. Figure 3.1 shows the mathematical methods used in it to achieve succinctness, anti-counterfeiting, non-interaction and so on (Suppose A is the prover and B is the verifier).
<img width="648" alt="principle" src="https://github.com/YunxiaShi/PHBS_BlockChain_2018/blob/master/principle.png">

Figure 3.1 The principle of zk-SNARK


* #### Describe the problem as a QAP
As a mathematical method, zk-SNARK must have quantifiable input, so we must first establish a mathematical model for the target problem. If the calculation equation corresponding to the target problem holds when the particular input values are substituted into it, then these inputs are called the "solutions" of the problem. What zk-SNARK has to deal with is to prove "I know the solution."

zk-SNARK is only suitable for a specific form of computational problem, the so-called QAP (Quadratic Arithmetic Programs), which writes the problem into a polynomial equation: `t(x) h(x) = w(x) v(x)`. A needs to verify that he knows the solution to this equation without showing the solution.

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

Thus, the homomorphic hiding problem of multiplication can be solved. The specific principles are as follows.

1. B selects point s randomly, calculates `E(s)`, `E(s^2)`,..., and sends it to A.
2. A calculates `E(t(s))`, `E(h(s))`, `E(w(s))`, `E(v(s))`.
3. B tests `E(t(s)h(s))==E(w(s)v(s))`.

* #### KCA (Knowledge of Coefficient Test and Assumption)
There is a problem with the above verification method. B cannot verify that A is actually using the polynomial `t(s)`, `h(s)`, `w(s)`, `v(s)` to calculate the result, that is, it cannot be proved that A really knows these polynomials. KCA can solve this problem.

Let us first define a concept: `α` pair refers to a pair of values `(a, b)` that satisfy `b = α * a`. Note that the multiplication here is actually the multiplication on the elliptic curve (ECC). The operation on the elliptic curve has two characteristics: First, when the value of `α` is large, it is difficult to guess `α` through `a` and `b`. And multiplication of elliptic functions satisfies homomorphic hidden.

We use the characteristics of α pair to build a process called KCA:
1. B randomly selects an `α` to generate `α` pair `(a, b)`, saves `α` itself and sends `(a, b)` to A.
2. A selects `γ`, generates `(a', b') = (γ⋅a, γ⋅b)`, and returns `(a', b')` to B. Using the commutative law, it can be proved that `(a', b')` is also an `α` pair, `b' = γ⋅b = γα⋅a = α(γ⋅a) = α⋅a'`.
3. B checks `(a', b')`, confirming that it is an `α` pair, it can be asserted that A knows `γ`.

The above process can be used to build a complete zk-SNARK verification process.
1. With the homomorphic hiding of multiplication, A and B can jointly select `x⋅g` as `E(x)`.
2.B calculates `g`, `s⋅g`,..., `s^d⋅g` and `α⋅g`, `αs⋅g`,...,`αs^d⋅g` and sends it to A.
3.A calculates `a=t(s)⋅g`, `b=αt(s)⋅g` and returns it to B.
4. The value of `a` is the `E(v(s))` result of the B required to be verified, and KCA guarantees that the value of `a` must be generated by polynomial.

* #### CRS (common reference string): reduce interation
The ideal situation is that A puts the evidence as a string on the chain and anyone can verify the conclusion. In fact, this strict zero-interaction proof has proven to be incapable of satisfying all proof scenarios. We are second to none, using a method called CRS to put the random numbers `α` and `s` in the "system".

The final zk-SNARK verification process is:

1. Configure `α` and `s` to calculate `(E1(1), E1(s),..., E1(s^d), E2(α), E2(αs),...,E2(αs^d))`, And publicize.
2.A uses the public parameter to calculate the verification polynomial.
3.B check the polynomial.

The random parameters built into the "system" are very important. Those who know the secret parameters have the authority of the super administrator and can create counterfeit coins at will. In fact, ZeroCash selects six trusted people from all over the world, each of which generates a part of the key. The six people's passwords are spliced together to generate the publicized data, and then the keys of each people are destroyed. No one has the super administrator's authority unless the six collude to cheat.

### 4. Conclusion
The decentralization, safety, tamper resistance, and traceability of blockchain technology have attracted wide attention and applied to various fields. However, blockchain is a distributed ledger technology. In order to quickly reach a consensus on the entire network node, its transactions are open and transparent, which poses a serious threat to users' privacy. How to protect user privacy on the blockchain has always been the focus of researchers. This paper first analyzes the data privacy issues currently faced by blockchain technology. Secondly, it introduces the mixing mechanism and cryptography technology in the existing privacy protection scheme. Finally, it comprehensively analyses ZeroCash and zk-SNARK technology. However, these technologies all have their own advantages and disadvantages, it is necessary to study more secure and efficient blockchain data privacy protection methods in the future.
