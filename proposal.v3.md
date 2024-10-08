![](./ebook.webp)

### Introduction

The first-sale doctrine has been a cornerstone of U.S. copyright law since 1908. This "doctrine", which gives copyright holders the right of "first-sale" of their work, has had the difficult task of balancing the interests of copyright holders and the interest of the public for over a century. 

And some would say that it has done a pretty good job. 

It has provided content creators and copyright holders with an enforcable means of being compensated for their hard work and it has facilitated the resale, rental, and lending of legally obtained books and other copyrighted works to the public without requiring the express consent of the publisher to do so. 

Unfortunately, the advent of digital technology and the internet has posed new challenges to this doctrine. With a physical book, when you loan it out it is no longer available for anyone else to read. Ebooks are quite different. Copies made of an eBook file will all be identical; no fuzzy letters or generational grain. Each copy is as good as the first. 

This leads to a problem. 

If we treated ebooks just like books, any file provided to you would be a copy. How would you go about verifying that your ebook was actually *deleted* from your computer once you returned the book?

So the publishers in all of their wisdom (and efforts to maximize profits), began **LICENSING** ebooks instead of selling you a copy of the work. When you pay for an ebook, you only have the right to read that ebook, usually on a single device, with no guarantee of future availability. Publishers have been raising the licensing fees charged to libraries and individuals so excessively in recent years that Congress is now investigating the practice. 

The publishers are using the digital distribution of ebooks to make an end-run around a doctrine that has been providing a balance between making money and the public good.

This document provides a quick history of the first-sale doctrine, its importance and then outlines a technical solution for maintaining first-sale doctrine dynamics for eBooks using blockchain and smart contract technology, specifically through the eBook checkout system defined below. By utilizing a system where an eBook can be loaned to one patron at a time, just like a physical book, we hope to help maintain existing dynamics of copyright law in the digital era.

### History of the First-Sale Doctrine

The first-sale doctrine originated in the early 20th century with the Supreme Court case **Bobbs-Merrill Co. v. Straus** (1908). The court ruled that once a copyrighted item is sold, the copyright owner no longer controls its distribution. This doctrine was codified in the Copyright Act of 1976, allowing the lawful owner of a copy of a work to sell, lend, or otherwise dispose of that copy without permission from the copyright holder.

### Importance of the First-Sale Doctrine

The monumental impact that the First-Sale Doctrine has had on the dissemination of knowledge to the benefit of humanity cannot be overstated. The following short list touches on just a few of the most important benefits:

1. #### Enhancing Accessibility and Affordability

The first-sale doctrine has played a crucial role in enhancing the accessibility and affordability of copyrighted works. It enabled the establishment of libraries, second-hand bookstores, and rental services, allowing people to access works without purchasing them new. This has been particularly beneficial for educational institutions and low-income individuals.

2. #### Fostering a Secondary Market

The doctrine has also fostered a robust secondary market for books, music, movies, and other media. This market has provided additional revenue streams for sellers and wider choices for consumers. It has allowed for the circulation of out-of-print works, preserving cultural and intellectual heritage.

3. #### Balancing Interests

By limiting the control of copyright holders over the distribution of individual copies after the initial sale, the first-sale doctrine has balanced the interests of creators and the public. It has ensured that once a work is legally purchased, the purchaser has the right to use and distribute it as they see fit, within the confines of the law.

### Challenges in the Digital Era

With the rise of digital technology, the first-sale doctrine faces significant challenges. Digital works, unlike physical copies, can be copied and distributed with ease, leading to potential widespread unauthorized dissemination. Technological protection measures and digital rights management (DRM) systems have been employed to control access and distribution, but these measures often limit legitimate secondary uses of digital works.

* #### Impact of Digital Networks

Digital networks enable the distribution of works without transferring ownership of a physical copy, complicating the application of the first-sale doctrine. The ease of copying digital works has led copyright holders to retain more control over distribution, often at the expense of consumer rights and secondary markets.

* #### Legal and Technological Measures

The Digital Millennium Copyright Act (DMCA) of 1998 further complicated the landscape by providing legal support for technological protection measures. While these measures help prevent unauthorized copying, they also restrict lawful secondary uses and diminish the applicability of the first-sale doctrine in the digital realm.

### Blockchain eBook Checkout Proposal: A Modern Solution

To address these challenges, our blockchain eBook checkout proposal leverages the security and transparency of blockchain technology for asset tracking and smart contracts for managing the lending process, specifically using the Stellar network and Soroban smart contracts. This system aims to uphold the principles of the First-Sale Doctrine in the digital age.

#### Creating and Issuing Digital Assets

We propose creating a digital asset for each eBook on the Stellar network, with custom properties to manage its lending status. This ensures that each digital copy can be uniquely identified and tracked. Asset creation on Stellar mainnet is not free. Using the XLM price of $0.08908 on July 11, 2024, it would cost about $0.34 to create the asset with two custom properties on mainnet. This fee acts as means of keeping asset spam off the mainnet as well as paying for compute and network costs. The option of creating a book-centric network does exist where libraries provide nodes if cost is an issue.

### Instructions for Adding a Stellar Asset for a Book

#### Step 1: Set Up the Stellar Development Environment

1. **Install Node.js and npm**:
   - Download and install Node.js from the official [website](https://nodejs.org/).
   - Verify the installation by running `node -v` and `npm -v` in your terminal.

2. **Install the Stellar SDK**:
   - Run the following command to install the Stellar SDK:
     ```sh
     npm install stellar-sdk
     ```

#### Step 2: Create a Keypair for the Issuing Account

Generate a keypair for the issuing account that will create and manage the book assets.

```javascript
const { Keypair } = require('stellar-sdk');

// Generate a keypair for the issuing account
const issuingKeypair = Keypair.random();

console.log('Issuing Account Keypair:');
console.log('Public Key:', issuingKeypair.publicKey());
console.log('Secret Key:', issuingKeypair.secret());
```

#### Step 3: Fund the Issuing Account

Use the Stellar testnet friendbot to fund the issuing account.

```javascript
const fetch = require('node-fetch');

async function fundAccount(publicKey) {
    const response = await fetch(`https://friendbot.stellar.org?addr=${publicKey}`);
    const responseJSON = await response.json();
    console.log('Friendbot response:', responseJSON);
}

fundAccount(issuingKeypair.publicKey());
```

#### Step 4: Create and Issue the Book Asset

Create a new asset representing the book and issue it on the Stellar network.

```javascript
const { Server, TransactionBuilder, Operation, Asset, Networks } = require('stellar-sdk');

// Set up the Stellar server
const server = new Server('https://horizon-testnet.stellar.org');

// Create a new asset representing the book
const bookAsset = new Asset('BookToken', issuingKeypair.publicKey());

async function createBookAsset() {
    // Load the issuing account
    const account = await server.loadAccount(issuingKeypair.publicKey());

    // Create the transaction to issue the asset and set the custom properties
    const transaction = new TransactionBuilder(account, {
        fee: await server.fetchBaseFee(),
        networkPassphrase: Networks.TESTNET,
    })
        .addOperation(Operation.changeTrust({
            asset: bookAsset,
            source: issuingKeypair.publicKey(),
        }))
        .addOperation(Operation.payment({
            destination: issuingKeypair.publicKey(),
            asset: bookAsset,
            amount: '0.0000001', // Minimum amount to create the asset
        }))
        .addOperation(Operation.manageData({
            name: 'isCheckedOut',
            value: 'false',
            source: issuingKeypair.publicKey(),
        }))
        .addOperation(Operation.manageData({
            name: 'checkedOutBy',
            value: '',
            source: issuingKeypair.publicKey(),
        }))
        .setTimeout(100)
        .build();

    // Sign and submit the transaction
    transaction.sign(issuingKeypair);
    await server.submitTransaction(transaction);

    console.log('Book asset created with isCheckedOut and checkedOutBy properties set');
}

createBookAsset();
```

### Step 5: Create a Soroban Smart Contract to Manage the `isCheckedOut` Property and Store the Request Sender Address

We then create a Soroban smart contract to manage the `isCheckedOut` property of the book asset and store the address of the request sender.

```rust
#![no_std]

use soroban_sdk::{contractimpl, Env, Symbol, Address, Bytes, BytesN, Timestamp};

pub struct LibraryContract;

#[contractimpl]
impl LibraryContract {
    pub fn check_out_book(env: Env, book_id: Symbol, user: Address, return_date: Timestamp) -> BytesN<32> {
        let is_checked_out = env.storage().has(&book_id);

        if is_checked_out {
            let checked_out_status: bool = env.storage().get(&book_id).unwrap().unwrap();
            if checked_out_status {
                panic!("Book is already checked out");
            }
        }

        // Update the status to checked out
        env.storage().set(&book_id, &true);

        // Store the address of the user who checked out the book
        let checked_out_by_key = Symbol::from_str("checkedOutBy");
        env.storage().set(&checked_out_by_key, &user);

        // Store the return date
        let return_date_key = Symbol::from_str("returnDate");
        env.storage().set(&return_date_key, &return_date);

        // Generate a new token (e.g., a unique identifier for this checkout session)
        let new_token = env.random().generate();
        let token_key = Symbol::from_str("checkoutToken");
        env.storage().set(&token_key, &new_token);

        new_token
    }

    pub fn return_book(env: Env, book_id: Symbol, user: Address) {
        let is_checked_out = env.storage().has(&book_id);

        if !is_checked_out {
            panic!("Book does not exist");
        }

        let checked_out_status: bool = env.storage().get(&book_id).unwrap().unwrap();
        if !checked_out_status {
            panic!("Book is already returned");
        }

        // Verify the user returning the book
        let checked_out_by_key = Symbol::from_str("checkedOutBy");
        let stored_user: Address = env.storage().get(&checked_out_by_key).unwrap().unwrap();
        if stored_user != user {
            panic!("Unauthorized return");
        }

        // Update the status to returned
        env.storage().set(&book_id, &false);

        // Clear the checkout token
        let token_key = Symbol::from_str("checkoutToken");
        env.storage().remove(&token_key);
    }

    pub fn verify_token(env: Env, token: BytesN<32>) -> bool {
        let token_key = Symbol::from_str("checkoutToken");
        let stored_token: BytesN<32> = env.storage().get(&token_key).unwrap().unwrap();
        stored_token == token
    }

    pub fn invalidate_token(env: Env, book_id: Symbol) {
        let return_date_key = Symbol::from_str("returnDate");
        let stored_return_date: Timestamp = env.storage().get(&return_date_key).unwrap().unwrap();
        
        if env.now() > stored_return_date {
            // Update the status to returned
            env.storage().set(&book_id, &false);

            // Clear the checkout token
            let token_key = Symbol::from_str("checkoutToken");
            env.storage().remove(&token_key);
        }
    }
}
```

### Step 6: Encrypt the eBook with the Generated Token

We use the generated token to encrypt the eBook. This ensures that only the user with the valid token can decrypt and read this particular copy of the eBook. 

This code runs in a node.js environment and must have access to the unencrypted ebook file.

```javascript
const crypto = require('crypto');

// Encrypt the eBook
function encryptEBook(eBookContent, token) {
    const algorithm = 'aes-256-ctr';
    const cipher = crypto.createCipheriv(algorithm, token, Buffer.alloc(16, 0));
    const encrypted = Buffer.concat([cipher.update(eBookContent), cipher.final()]);
    return encrypted;
}

// Decrypt the eBook
function decryptEBook(encryptedEBook, token) {
    const algorithm = 'aes-256-ctr';
    const decipher = crypto.createDecipheriv(algorithm, token, Buffer.alloc(16, 0));
    const decrypted = Buffer.concat([decipher.update(encryptedEBook), decipher.final()]);
    return decrypted;
}

// Example usage
const eBookContent = Buffer.from('This is the content of the eBook');
const token = crypto.randomBytes(32); // This should be the token generated by the smart contract

const encryptedEBook = encryptEBook(eBookContent, token);
console.log('Encrypted eBook:', encryptedEBook);

const decryptedEBook = decryptEBook(encryptedEBook, token);
console.log('Decrypted eBook:', decryptedEBook.toString());
```

### Step 7: Integrate with the Smart Contract

1. **Checkout**:
   - Call the `check_out_book` function to get the token.
   - Use the token to encrypt the eBook.

2. **Return**:
   - Call the `return_book` function to invalidate the token.

3. **Verification**:
   - Call the `verify_token` function to check if the token is valid before allowing the user to decrypt and read the eBook.

4. **Auto-Invalidate**:
   - Periodically call the `invalidate_token` function to check if the return date has passed and invalidate the token if necessary.

### Project Deliverables

The following list of items will be complete at 4 weeks:

1. **Create accounts**:
  - Create reference 'librarian' account on Stellar network. This account will own the assets.
  - Create example 'user' account on Stellar for demonstration purposes.
2. **Create Soroban Smart Contract for eLoan**:
  - Soroban Smart Contract created as defined.
3. **Add 5 assets(books) to blockchain**:
  - Setup five(5) eBooks on Stellar blockchain for testing
4. **Create and deploy node.js Checkout Server**:
  - Deliver requested ebook to user
    - Lookup asset on blockchain
    - Encrypt ebook file with checkout token
    - Deliver encrypted ebook file to user
  - Handle book returns 
    - Invalidate token/expire keys
    - Update asset status on blockchain
5. **Document and Demonstrate end-to-end Checkout Process**
  - Clear documentation of checkout process
  - Simple demonstration of ebook checkout from start to finish

### TL;DR

For the past 116 years, the "First-Sale Doctrine" of U.S. copyright law has defined what rights are granted for books and other copyrighted media. 

The digital age, however, poses significant challenges to this doctrine. Our blockchain eBook checkout proposal offers a modern solution, using the Stellar blockchain for tracking assets (books/ebooks) and Soroban smart contracts to ensure secure and controlled lending of digital works. By adapting the first-sale doctrine to the digital era, we can preserve its benefits and continue to promote access to knowledge and culture.

### Future Work

Further work could involve:
  
- Integrating the system with existing library management software.
- Developing user-friendly interfaces for library staff and patrons.
- Implementing additional features such as notifications for overdue books and automated extensions.

This proposal aims to leverage cutting-edge blockchain technology to modernize library operations and improve the accessibility and security of digital resources. We look forward to the opportunity to implement and demonstrate this innovative solution.
