Q1. A blockchain is a distributed ledger operating over a Peer-to-Peer (P2P) network. Data in a blockchain is organized in discrete blocks chained together cryptographically, hence the name. Each block contains transactional data, a timestamp and the hash digest of the contents of the previous block in the sequence. This digest acts as a unique fingerprint that "locks" all the data in the chain up to that point by exploring the mathematical properties of the hash function. If even one bit of data in any of the previous blocks changes, the hash digest of the last block of the chain also changes but in a dramatic and unpredictable fashion, which makes blockchain forgery a probabilistically challenging task, i.e., almost impossible.

Q2. Smart contracts are software scripts, written in a specific programming language, that can be used to abstract and automate blockchain related tasks. Smart contracts are inherently open source because, once deployed into a public blockchain, their code is written into a block that can be consulted by anyone. To protect the Virtual Machine where smart contracts are executed from malicious or unoptimized code, each instruction requires gas to execute. So gas needs to be bough beforehand by the contract executor and its often bought using the cryptocurrency supported by the blockchain, therefore, running smart contracts costs money. Once deployed, a smart contract can be run executing a transaction from an Externally Own Wallet or another smart contract indicating its deployment address, providing enough funds for its execution (gas) and any arguments required by the function to execute. Because data in a blockchain is immutable, Smart Contracts cannot be edited after deployment. To change a deployed smart contract, one has to deploy the new version to a new block and publish the new contract address. Ideally one should also delete the previous contract (which can be done by writing specific a self-destruct function into it) to keep blockchain garbage code to a minimum, but its often not a requirement.

Q3. A script is a sequence of instructions. A computer program and a cooking recipe are examples of scripts. A transaction is the action of transfering a quantity between two or more entities. In a blockchain context, a transaction indicates how much of a certain cryptocurrency (depending on the underlying blockchain) is tranferred between wallet addresses. Since Smart Contracts are great examples of blockchain-specific scripts and these are executed via transactions, in a blockchain scripts (smart contracts) can be used to execute transactions and transactions are used to execute scripts.