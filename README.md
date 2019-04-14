# PHBS_BlockChain_2018

## ZeroCash and sk-SNARK

### 1. Zero knowledge proof
Zero knowledge proof was proposed by Goldwasser in 1989 and was a two-party or multi-party agreement. It allows one party (the prover) to prove to another (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself. Take the currency transaction as an example. It means the transaction can be proved valid without revealing the payer, the payee, or the transaction amount.

In 2013, based on zero-knowledge proof, [Miers](https://ieeexplore.ieee.org/document/6547123) proposed ZeroCoin, which realized a completely anonymous digital currency transaction, but there are still many problems with ZeroCoin. For example, it does not support non-interactive transactions, does not protect the transaction amount and the privacy of the payee, and the currency amount can not be arbitrarily divided. To this end, many new schemes have been proposed. The ZeroCash proposed by [SASSON](https://ieeexplore.ieee.org/document/6956581) is the most typical.

### 2. [How ZeroCash works](https://blog.z.cash/zcash-private-transactions/)
Based on ZeroCoin, ZeroCash adopts zk-SNARK technology, which can effectively protect the information of the payer, payee and the payment amount, and achieves full anonymity. This part is to provide a simplified explanation of how privacy-preserving transactions work in ZeroCash, and where exactly Zero knowledge proof comes into the picture.

In Bitcoin, UXTO is the basic transaction unit, and ZeroCash uses note as the basic transaction unit. Simplify the note as `note = (PK, v, r)`, PK is the owner’s public key (address), v is the amount, and r is the serial number that can uniquely distinguish the note. 

The transactions in ZeroCash have two categories, transparent addresses and hidden addresses. The input and output of the transparent address transaction are directly visible note information. For hidden address transactions, the input and output are no longer plaintext notes, but the note nullifier and note commitment.

* Note commitment

It works as an output of the transaction and indicates a new note has been issued. A valid commitment is a proof of a spendable note. However, we need to make sure that the information it contains does not reveal which note it is, who the owner is and how much amount it is. Therefore, we can hash the information of the note. And the commit corresponding to the note can be simply described as `HASH(note)`.

* Note nullifier

It works as an input of the transaction and indicates an old note will be void. Like Bitcoin, the input of one transaction must be the output of another transaction. So the nullifier corresponds to a commitment uniquely, but it should avoid disclosing any information about which commitment the nullifier is related to. To construct a nullifier that satisfies the requirements, it is still a good idea to take a hash. Therefore, the nullifier corresponding to the note with the serial number r can be described as `HASH(r)`.

Bitcoin tracks unspent transaction outputs (UTXOs) to determine what transactions are spendable. In ZeroCash, the shielded equivalent of a UTXO is called a “commitment”, and spending a commitment involves revealing a nullifier. ZeroCash nodes keep lists of all the commitments that have been created and all the nullifiers that have been revealed.

Suppose there are three notes currently, note1=(PK1,v1,r1)，note2=(PK2,v2,r2)，note3=(PK3,v3,r3). The note1 belongs to Anna, and note2 has been spent. The contents of The nullifier and commitment list maintained by each node at this time is shown in Table 1.1.

Commitment Set,Nullifier Set
`C1 = HASH(note1)`,`NF1 = HASH(r2)`pku,04e284716af198b11fe56a302cbc899631364d16be4db2c321f235e884c084f6badb61cd398cea61cf46af24f9c066cbdfc244638ad893c92b8d0c58ff3ba333ad

