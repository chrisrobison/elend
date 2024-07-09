## Utilizing Stellar Blockchain and Soroban Smart Contracts for Library eBook Lending

### Introduction

For the past 116 years, U.S. Copyright Law has provided the "First-Sale" doctrine for copyrighted works. This allows the owner of a legally acquired book (or record, CD, etc.) to loan, sell or rent their copy of the copyrighted work. Copyright law attempts to provide a balance between allowing content authors to monetize their work and ensuring public access to that work.  
Libraries face the challenge of managing digital assets and intellectual property rights efficiently while ensuring that eBooks are accessible to users in a secure manner. Blockchain technology, with its inherent security and transparency, offers a solution to these challenges. By leveraging the Stellar blockchain and Soroban smart contracts, we can create a system for managing eBook lending that ensures only one person can read an eBook at a time and that access is revoked once the lending period ends.

### Objectives

1. Develop a system to manage eBook lending using Stellar blockchain.
2. Ensure that eBooks can only be checked out by one person at a time.
3. Revoke access to eBooks once they are returned or the lending period expires.
4. Utilize Soroban smart contracts to automate and secure the lending process.

### Technical Implementation

#### Step 1: Add an Asset (Book) with Custom Boolean Property `isCheckedOut` and Address Property `checkedOutBy`

We first create an asset on the Stellar network representing the book and add custom properties to track its lending status.

```javascript
const { Keypair, Server, TransactionBuilder, Operation, Asset, Networks } = require('stellar-sdk');
const fetch = require('node-fetch');

// Set up the Stellar server
const server = new Server('https://horizon-testnet.stellar.org');

// Generate a keypair for the issuing account
const issuingKeypair = Keypair.random();

// Create a new asset representing the book
const bookAsset = new Asset('BookToken', issuingKeypair.publicKey());

// Function to create the asset and set the custom properties
async function createBookAsset() {
    // Fund the issuing account
    await fetch(`https://friendbot.stellar.org?addr=${issuingKeypair.publicKey()}`);

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

#### Step 2: Create a Soroban Smart Contract to Manage the `isCheckedOut` Property and Store the Request Sender Address

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

### Step 3: Encrypt the eBook with the Generated Token

We use the generated token to encrypt the eBook. This ensures that only the user with the valid token can decrypt and read the eBook.

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

### Step 4: Integrate with the Smart Contract

1. **Checkout**:
   - Call the `check_out_book` function to get the token.
   - Use the token to encrypt the eBook.

2. **Return**:
   - Call the `return_book` function to invalidate the token.

3. **Verification**:
   - Call the `verify_token` function to check if the token is valid before allowing the user to decrypt and read the eBook.

4. **Auto-Invalidate**:
   - Periodically call the `invalidate_token` function to check if the return date has passed and invalidate the token if necessary.

### Conclusion

By leveraging the Stellar blockchain and Soroban smart contracts, we can create a secure and efficient system for managing eBook lending in libraries. This system ensures that only one person can read an eBook at a time, and access is revoked once the book is returned or the lending period expires. This approach not only enhances the security and transparency of the lending process but also aligns with the goals of preserving intellectual property rights and providing seamless access to digital resources.

### Proposed Work

Proposed work will involve:

1. Implementing and deploying Checkout contract
2. Working with an existing library to import assets (physical and/or digital)
3. Test and fine-tune the blockchain-based checkout process

The library I had in mind for testing purposes is the [Internet Archive](https://internetarchive.org) as they have an outstanding collection of books, both physical as well as scanned ebooks. The Internet Archive has been in the news recently due to being sued by publishers for over-loaning scanned books.

### Future Work

Future work could involve:

1. Integrate the system with existing library management software.
2. Developing user-friendly interfaces for library staff and patrons.
3. Implementing additional features such as notifications for overdue books and automated extensions.

This proposal aims to leverage cutting-edge blockchain technology to modernize library operations and improve the accessibility and security of digital resources. We look forward to the opportunity to implement and demonstrate this innovative solution.
