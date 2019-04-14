# PHBS_BlockChain_2018

## Zerocash and sk-SNARK

### 1. Zero knowledge proof
Zero knowledge proof was proposed by Goldwasser in 1989 and was a two-party or multi-party agreement. It allows one party (the prover) to prove to another (the verifier) that a statement is true, without revealing any information beyond the validity of the statement itself. Take the currency transaction as an example. It means the transaction can be proved valid without revealing the payer, the payee, or the transaction amount.

In 2013, based on zero-knowledge proof, Miers proposed ZeroCoin [12], which realized a completely anonymous digital currency transaction, but there are still many problems with ZeroCoin. For example, it does not support non-interactive transactions, does not protect the transaction amount and the privacy of the payee, and the currency amount can not be arbitrarily divided. To this end, many new schemes have been proposed. The ZeroCash proposed by SASSON [14] is the most typical.

