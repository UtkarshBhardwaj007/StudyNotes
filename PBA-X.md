# 1. Cryptography
## 1.1 Kerckhoff's Principle:
A cryptographic system should be secure even if everything about the system, except the key, is public knowledge.

## 1.2 Hashing attacks
* **Preimage Attack**: Given a hash value h, find any message m such that `hash(m) = h`. Trying to find an input that produces a specific target hash
* **Weak Collision Attack (Second Preimage Attack)**: Given a message `m1`, find another message `m2` such that `hash(m1) = hash(m2)`.
* **Strong Collision Attack**: Find any two different messages `m1` and `m2` such that hash(m1) = hash(m2). Can use any two inputs, not tied to a specific input or hash.

## 1.3 Encryption
* Symmetric key encryption is less computationally expensive, so faster, than asymmetric key encryption. 
* So we try to share the symmetric keys securely and then use them for communication. 2 ways to share keys:
    * **Diffie Hellman key exchange**: Both parties involved create unique secret keys. Then they share a common `base` and `modulus` and use modular arithmetic to calculate a shared secret key.
    * **Asymmetric Key Encryption (RSA)**: Here both parties create separate public and private keys. The public key is shared with the other party and the private key is kept secret. Both parties use this secure channel to share a symmetric key.

## 1.4 Wallet and Accounts
* A crypto wallet doesn't store any crypto assets. It stores the private keys which prove ownership of the crypto assets.
* The public key is the wallet address.
* Pneumonic phrases are a way to generate a private key from a random string of words.
* We can't remember a `2048` bit private key, so we use a pass phrase to generate it. The magic here is that we change the base of the private key from `2` to `58` (in case of polkadot) so that the private keys becomes shorter and is easy to remember.
* The specific address format used in polkadot is `SS58` format. `SS58` is a base58 encoding of the public key and it adds `2` bytes at the beginning to "indicate the network". It also adds `2` bytes at the end for `Checksum`.
* **Hard Derived Account**: Has it's own private key and public key. Created with `\\` (2 slashes).
* **Soft Derived Account**: Has it's own public key and shares the private key with the parent account. Created with `/` (1 slash).

## 1.5 Digital Signatures
* A digital signature is a mathematical scheme for verifying the authenticity of a digital message or document.
* Guaranteed properties of a Digital Signature scheme in the context of transactions:
    * **Authenticity**: a valid signature implies that the signer deliberately signed the transaction.
    * **Unforgeability**: it is computationally infeasible to forge a signature without knowledge of the private key.
    * **Non-repudiation**: the signer cannot later deny having signed the message
    * **Integrity**: ensure the transaction data has not been modified
* It consists of 3 algorithms:
    * **Key Generation**: Generate a public and private key pair.
    * **Signing Algorithm**: Uses the private key and the message to create a signature.
    * **Verification Algorithm**: Uses the public key and the original message and the signature to verify if the signature is valid. The signature will only verify correctly if the message hasn't been tampered with and the public key corresponds to the private key that was used to create the signature.
* **In Practice** digital signatures don't sign the entire message. Instead a hash function is used to hash the message and create a `Digest` or `Fingerprint` of fixed length. The reason for this is that Cryptographic hashing functions usually work on fixed size inputs.
* One challenge that Digital signatures address is `Replay Attacks`. A replay attack is when an attacker captures a message and then later re-sends it to the recipient. The recipient can't tell if the message is new or old. To prevent this, Digital signatures often use additional information like `nonces` and `timestamps`/`lifetimes`.
* **Multisig accounts** are accounts that require multiple signatures to authorize a transaction. This is useful for things like other decentralized organizations.

## 1.6 Hash Based Data Structures

### 1.6.1 Hash Chains
* A hash chain is a fundamental data structure used in blockchains to ensure the integrity and immutability of data. It is a sequence of blocks (objects), each containing data and a cryptographic hash of the previous block, forming a chain (could be anything like vector or list). This structure ensures that any alteration in a block would require changes to all subsequent blocks, making it tamper-evident.
* Most blockchain implementations (including major ones like Bitcoin) store blocks in a database or vector and use the hash values for validation, not traversal.

### 1.6.2 Merkle Trees
* Merkle tree also known as hash tree is a data structure used for data verification and synchronization. 
* It is a tree data structure where each non-leaf node is a hash of it’s child nodes. All the leaf nodes are at the same depth and are as far left as possible. 
* Each transaction gets hashed, Hashes are paired and hashed again, Process repeats until single root hash, Root hash goes in block header of the blockchain block.
```mermaid
graph TD
    subgraph "Block Header"
        MR[Merkle Root: H1234]
    end
    
    subgraph "Merkle Tree"
        H1234[H1234: Root Hash] --- H12[H12]
        H1234 --- H34[H34]
        
        H12 --- H1[H1]
        H12 --- H2[H2]
        H34 --- H3[H3]
        H34 --- H4[H4]
        
        H1 --- Tx1[Transaction 1: Alice sends 5 BTC to Bob]
        H2 --- Tx2[Transaction 2: Bob sends 2 BTC to Charlie]
        H3 --- Tx3[Transaction 3: Dave sends 1 BTC to Alice]
        H4 --- Tx4[Transaction 4: Charlie sends 0.5 BTC to Dave]
    end
```
### 1.6.3 Merkle Mountain Range
* A Merkle Mountain Range (MMR) is a variation of Merkle Trees that allows efficient appending of new elements. Think of it as a collection of perfect binary trees of different heights, forming a "mountain range" profile.
* Efficient appending O(log n), Easy to prove membership, Good for growing datasets, Used in blockchain UTXO sets.

```mermaid
graph TD
    subgraph "UTXO Management with MMR"
        Root[MMR Root] --- P1[Peak 1]
        Root --- P2[Peak 2]
        
        P1 --- U1[UTXOs 1+2]
        P1 --- U2[UTXOs 3+4]
        
        P2 --- U3[UTXO 5]
        
        U1 --- TX1[5 BTC UTXO]
        U1 --- TX2[3 BTC UTXO]
        U2 --- TX3[1 BTC UTXO]
        U2 --- TX4[2 BTC UTXO]
        U3 --- TX5[0.5 BTC UTXO]
        
        style Root fill:#f9f,stroke:#333
        style P1 fill:#bbf,stroke:#333
        style P2 fill:#bbf,stroke:#333
        style TX1 fill:#bfb,stroke:#333
        style TX2 fill:#bfb,stroke:#333
        style TX3 fill:#bfb,stroke:#333
        style TX4 fill:#bfb,stroke:#333
        style TX5 fill:#bfb,stroke:#333
    end
```
* MMR provides a way for blockchain UTXO management. When new transaction creates UTXOs:
    * Hash the new UTXOs
    * Add to MMR structure
    * Update peaks if needed
* When spending UTXOs:
    * Prove UTXO exists using MMR path
    * Remove spent UTXO
    * Add new UTXOs from transaction

### 1.6.4 Radix/Patricia Tries
* A Patricia Trie (Practical Algorithm To Retrieve Information Coded In Alphanumeric) is a modified trie that compresses nodes with only one child, making it more space-efficient. In blockchains, it's used to store state data efficiently.
```mermaid
graph TD
    subgraph "Patricia Trie State Storage"
        SR[State Root Hash] --> A1[Address: 0xa...]
        SR --> A2[Address: 0xb...]
        
        A1 --> |Account State| AS1["Account A Data:
        - Balance: 100 ETH
        - Nonce: 5
        - Code Hash
        - Storage Root"]
        
        A2 --> |Smart Contract| AS2["Contract B Data:
        - Balance: 50 ETH
        - Nonce: 12
        - Code Hash
        - Storage Root"]
        
        AS2 --> |Storage Trie| ST["Contract Storage:
        Key1: Value1
        Key2: Value2
        ..."]
        
        style SR fill:#f96,stroke:#333,stroke-width:4px
        style A1 fill:#bbf,stroke:#333
        style A2 fill:#bbf,stroke:#333
        style AS1 fill:#bfb,stroke:#333
        style AS2 fill:#bfb,stroke:#333
        style ST fill:#fbb,stroke:#333
    end

    subgraph "Key Features"
        SR -.- K1["✓ Single Root Hash 
        verifies entire state"]
        AS1 -.- K2["✓ Efficient Proofs 
        for account data"]
        ST -.- K3["✓ Separate Storage Trie 
        for each contract"]
    end
```

## 1.7 Advanced Cryptography Topics

### 1.7.1 Shamir Secret Sharing
* Shamir Secret Sharing is a cryptographic method to split a secret into n shares, where any k shares (threshold) can reconstruct the secret, but k-1 or fewer shares reveal nothing about the secret.

### 1.7.2 Verifiable Random Functions (VRFs)
A VRF is a cryptographic function that:
* Takes an input and private key to generate a random number
* Provides a proof that the number was generated correctly
* Anyone can verify the proof using the public key
* Same input + private key always gives same output
* Output appears random to anyone without the private key

### 1.7.3 Zero-Knowledge Proofs (ZKPs)
* A zero knowledge proof is a method where one party (prover) can prove to another party (verifier) that a statement is true without revealing any information beyond the validity of the statement.

# 2. Fundamentals of blockchains

## 2.1 Blockchain Concepts Pre-Quiz:
* Bitcoin uses `UTXO` model instead of the `Accounts` model to process transactions.
* Typically, The hash of the NFT images and videos is stored on the blockchain network. The original content is stored typically on a decentralized storage platform which may or may not use blockchain technology stack. Storing them directly on the blocks or in the blockchain state can be expensive and is an inefficient implementation of the NFT use case.
* Just because the transaction shows up on Bitcoin block explorers does not mean the transaction is final. The finality of the Bitcoin blockchain is probabilistic. That is why central exchanges wait a couple of hours before letting you transact with the Bitcoin you deposited on their platform.
* The security guarantees offered by Ethereum Roll-ups (Layer 2 chains) are **NOT** the same as the security guarantees of the Ethereum network (Layer 1 chain).
* Blockchain network's security and resilience to attacks rely on factors like decentralization, consensus mechanisms, and protocol design.

## 2.2 Blockchain Networks:

### 2.2.1 Types of Decentralization
* **Architectural Decentralization**: To address single point of failures in the network (Distributed systems)
* **Political Decentralization**: Who gets to decide how the blockchain should operate? What is the governance model?
* **Logical Decentralization**: Blockchains may be politically decentralized (no one controls them) and architecturally decentralized (no infrastructural central point of failure) but they are logically centralized. There is one commonly agreed state and the system behaves like a single computer. Most blockchains are logically centralized.

## 2.3 Blockchain Architecture
* Blockchain architecture consists of multiple layers, each with specific functions:
    * **Application Layer**: Top-most layer where users interact with blockchain. Includes dApps, smart contracts, wallets, and user interfaces. Provides services and functionality to end users
    * **Consensus Layer**: Ensures all nodes agree on the state of the blockchain. Implements consensus mechanisms (PoW, PoS, etc.). Validates and orders transactions
    * **Network Layer**: Handles peer-to-peer communication between nodes. Manages transaction and block propagation. Maintains network connectivity and node discovery
    * **Data Layer**: Defines structure of blockchain data (blocks, transactions). Implements cryptographic primitives and data structures. Handles data storage and retrieval
    * **Infrastructure Layer**: Base layer providing hardware and software requirements. Includes nodes, physical networks, and computing resources. Supports basic blockchain operations
```mermaid
graph TD
    subgraph "Blockchain Architecture Layers"
        A[Application Layer] --> C
        C[Consensus Layer] --> N
        N[Network Layer] --> D
        D[Data Layer] --> I
        I[Infrastructure Layer]
        
        A -.- A1["dApps, Smart Contracts, 
        User Interfaces"]
        C -.- C1["PoW/PoS, 
        State Agreement"]
        N -.- N1["P2P Communication,
        Node Discovery"]
        D -.- D1["Blocks, Transactions,
        Cryptography"]
        I -.- I1["Nodes, Hardware,
        Base Network"]
        
        style A fill:#f96,stroke:#333
        style C fill:#bbf,stroke:#333
        style N fill:#bfb,stroke:#333
        style D fill:#fbb,stroke:#333
        style I fill:#ddd,stroke:#333
    end
```
* Different **types of blockchain architectures**:
```mermaid
graph TD
    subgraph "Blockchain Architectures"
        subgraph "Monolithic"
            M[Single Chain] --> M1[Consensus]
            M --> M2[Execution]
            M --> M3[Data]
            style M fill:#f96,stroke:#333
        end

        subgraph "Modular"
            MD[Base Layer] --> E[Execution Layer]
            MD --> D[Data Layer]
            MD --> S[Settlement Layer]
            style MD fill:#bbf,stroke:#333
        end

        subgraph "Multi-chain"
            MC[Main Chain] --> P1[Chain 1]
            MC --> P2[Chain 2]
            MC --> P3[Chain 3]
            style MC fill:#bfb,stroke:#333
        end
    end

    subgraph "Examples"
        M -.- ME[Ethereum 1.0]
        MD -.- MDE[Celestia]
        MC -.- MCE[Polkadot]
    end
```
* Layers in Blockchain Ecosystem Architecture:
    * **Layer 0**: Network infrastructure protocols that allow blockchains to be built and communicate
        * Not a blockchain itself
        * A foundation layer/protocol
        * Example: Polkadot isn't a blockchain, it's a protocol that allows creation and connection of blockchains (parachains)
    * **Layer 1**: Actual base blockchains
        * These ARE blockchains
        * Independent networks with their own consensus
        * Example: Ethereum is a L1 blockchain
    * **Layer 2**: Scaling solutions built ON TOP of L1 blockchains
        * Not independent blockchains
        * Solutions that extend L1 capabilities
        * Example: Optimism isn't its own blockchain, it's a scaling solution for Ethereum

## 2.4 Blockchain Trilemma
* The Blockchain Trilemma refers to the fundamental challenge in blockchain design where it's seemingly impossible to achieve all three key properties simultaneously:
    * Decentralization
    * Security
    * Scalability

## 2.5 Blockchain Consensus

### 2.5.1 Layers of Consensus
These are used in consensus mechanisms to filter out chains until we find the canonical chain.
* **State Machine Validity**: This checks if the history of transaction (`State Transitions`) are valid.
* **Political Validity**: We might reject blocks that don't meet certain `political` or `arbitrary` rules. For example, in Bitcoin, blocks bigger than 1MB are rejected.
* **Right to Author**: This checks if the block was authored by the right person who was elected to author the block.
* **Fork Choice**: Each node has its own view of the blockchain and decides, based on that, if the current block is valid or not. A block in an invalid fork would be rejected.

### 2.5.2 Consensus Mechanisms
* **Proof of Work (PoW)**: 
    * It is a `permissionless` consensus mechanism that requires miners to solve complex mathematical problems to validate transactions and add new blocks to the blockchain. It ties the blockchain's value to a real world scarce resource (energy).
    * Downsides: It is energy-intensive and the blocks are produced at varying times.
* **Proof of Stake (PoS)**:
    * A consensus mechanism where validators stake (lock up) cryptocurrency to get the right to validate blocks. The more you stake, the higher chance of being selected to validate.
* **NOTE**: `PoW` and `PoS` are mechanisms for `block authoring`. Consensus on `block finality` is achieved through finality gadgets like `GRANDPA` for deterministic block finality.

### 2.5.3 Block Finality Mechanisms

* **Block Finality**: The guarantee that a block cannot be reverted/changed once added to the chain.
* **Finality Gadgets**: These are mechanisms that allow a blockchain to reach `Deterministic Finality`. In Polkadot, the primary finality gadget is `GRANDPA`(GHOST-based Recursive Ancestor Deriving Prefix Agreement).

#### 2.5.3.1 Probabilistic Finality
* Finality becomes stronger over time. More blocks = more certainty. Used in BTC.
* There is always a risk of reverting a block if a longer chain is found.

#### 2.5.3.2 Deterministic Finality
* Finality is guaranteed. Once confirmed, block cannot be reverted. Used in PoS systems like Polkadot. Example: After 2/3 validators sign, block is final.
* There is no risk of reverting a block.

#### 2.5.3.3 GRANDPA
* GRANDPA is responsible for finalising blocks in polkadot.
* Polkadot uses a two-phase voting system (`Hybrid Consensus Model`) to reach finality. It separates the block production from block finality.

### 2.5.4 Forks in Blockchain
* **Soft Fork**: A change to the protocol that is backward compatible. Tightens/adds rules. Old nodes can still validate blocks.
* **Hard Fork**: A change to the protocol that is not backward compatible. Changes fundamental rules. Old nodes will reject new blocks.
