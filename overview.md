### Project Architecture

The **eLend** checkout system is built on a number of technologies to accomplish its goal lending "one ebook to one reader" at a time, respecting copyright law. This document outlines the technologies used and provides a little commentary on some of the benefits, drawbacks and reasoning behind the choices made.

### Stellar Blockchain

**Stellar** is an open-source, decentralized blockchain platform designed to facilitate fast, low-cost, cross-border transactions. Key features of the Stellar blockchain include:

- **Stellar Consensus Protocol (SCP)**: Unlike proof-of-work or proof-of-stake, SCP uses a federated Byzantine agreement (FBA) model, which allows for quicker consensus without the need for extensive energy consumption.
- **Lumens (XLM)**: The native currency of the Stellar network, used to facilitate transactions and prevent spam.
- **Asset Issuance and Transfer**: Stellar allows users to issue their own assets (tokens) and transfer them over the network. This makes it suitable for applications like remittances, micropayments, and tokenization of assets.
- **Decentralized Exchange (DEX)**: Built into the Stellar protocol, the DEX allows users to trade assets directly on the network without the need for a central exchange.

### Soroban Smart Contracts

**Soroban** is Stellar's smart contract platform, designed to bring more functionality and flexibility to the Stellar ecosystem. Key aspects of Soroban smart contracts include:

- **Turing-Complete Language**: Soroban is designed to support a Turing-complete language, enabling the creation of more complex and flexible smart contracts compared to previous capabilities on Stellar.
- **Interoperability**: Soroban smart contracts can interact with Stellar's existing features, such as asset issuance and the decentralized exchange, allowing for seamless integration with existing functionalities.
- **Performance and Scalability**: Soroban is optimized for high performance and scalability, ensuring that smart contracts can execute efficiently even under high transaction volumes.
- **Security**: Soroban places a strong emphasis on security, incorporating features to prevent common smart contract vulnerabilities and ensuring robust contract execution.

Soroban aims to expand Stellar's use cases by enabling more sophisticated applications, such as decentralized finance (DeFi), non-fungible tokens (NFTs), and more complex financial instruments.

