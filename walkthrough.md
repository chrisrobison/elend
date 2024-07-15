### Detailed Walkthrough of eBook Lending Using Stellar Blockchain and Soroban Smart Contracts

#### 1. Initial Process of Adding an eBook to the Stellar Blockchain as an Asset

**Step-by-Step Process:**

1. **Set Up the Stellar Development Environment**:
    - Install Node.js and the Stellar SDK.
    - Use the Stellar SDK to interact with the Stellar network.

2. **Create a Keypair for the Issuing Account**:
    - Generate a keypair for the issuing account, which will create and manage the book assets.
    ```javascript
    const { Keypair } = require('stellar-sdk');
    const issuingKeypair = Keypair.random();
    console.log('Public Key:', issuingKeypair.publicKey());
    console.log('Secret Key:', issuingKeypair.secret());
    ```

3. **Fund the Issuing Account**:
    - Use the Stellar testnet friendbot to fund the issuing account.
    ```javascript
    const fetch = require('node-fetch');
    async function fundAccount(publicKey) {
        const response = await fetch(`https://friendbot.stellar.org?addr=${publicKey}`);
        const responseJSON = await response.json();
        console.log('Friendbot response:', responseJSON);
    }
    fundAccount(issuingKeypair.publicKey());
    ```

4. **Create and Issue the Book Asset**:
    - Define and issue the digital asset representing the book on the Stellar network.
    ```javascript
    const { Server, TransactionBuilder, Operation, Asset, Networks } = require('stellar-sdk');
    const server = new Server('https://horizon-testnet.stellar.org');
    const bookAsset = new Asset('BookToken', issuingKeypair.publicKey());
    
    async function createBookAsset() {
        const account = await server.loadAccount(issuingKeypair.publicKey());
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
                amount: '0.0000001',
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
    
        transaction.sign(issuingKeypair);
        await server.submitTransaction(transaction);
    
        console.log('Book asset created with isCheckedOut and checkedOutBy properties set');
    }
    
    createBookAsset();
    ```

#### 2. The Checkout Process Flow

**a. Initial Request:**

1. A user requests to borrow an eBook.
2. The system receives the request and initiates the checkout process.

**b. Verifying Availability:**

1. The system checks the `isCheckedOut` status of the book asset on the Stellar blockchain.
    ```javascript
    async function checkBookAvailability(bookAsset) {
        const account = await server.loadAccount(issuingKeypair.publicKey());
        const isCheckedOut = account.data_attr['isCheckedOut'];
        return isCheckedOut === 'false';
    }
    ```

**c. Creating Token or Key for Encrypting Checked Out Book:**

1. If the book is available, the system generates a unique token using a Soroban smart contract.
    ```rust
    pub fn check_out_book(env: Env, book_id: Symbol, user: Address, return_date: Timestamp) -> BytesN<32> {
        let is_checked_out = env.storage().has(&book_id);

        if is_checked_out {
            let checked_out_status: bool = env.storage().get(&book_id).unwrap().unwrap();
            if checked_out_status {
                panic!("Book is already checked out");
            }
        }

        env.storage().set(&book_id, &true);
        let checked_out_by_key = Symbol::from_str("checkedOutBy");
        env.storage().set(&checked_out_by_key, &user);

        let return_date_key = Symbol::from_str("returnDate");
        env.storage().set(&return_date_key, &return_date);

        let new_token = env.random().generate();
        let token_key = Symbol::from_str("checkoutToken");
        env.storage().set(&token_key, &new_token);

        new_token
    }
    ```

**d. Updating Blockchain with New Info (User Address, Checkout Status):**

1. The smart contract updates the `isCheckedOut` status to `true`, stores the user’s address, and the return date.
    ```rust
    env.storage().set(&book_id, &true);
    env.storage().set(&checked_out_by_key, &user);
    env.storage().set(&return_date_key, &return_date);
    ```

**e. Delivery of Encrypted eBook File:**

1. The system encrypts the eBook using the generated token and delivers the encrypted eBook to the user.
    ```javascript
    const crypto = require('crypto');

    function encryptEBook(eBookContent, token) {
        const algorithm = 'aes-256-ctr';
        const cipher = crypto.createCipheriv(algorithm, token, Buffer.alloc(16, 0));
        const encrypted = Buffer.concat([cipher.update(eBookContent), cipher.final()]);
        return encrypted;
    }

    const eBookContent = Buffer.from('This is the content of the eBook');
    const token = /* token from the smart contract */;
    const encryptedEBook = encryptEBook(eBookContent, token);
    // Deliver the encrypted eBook to the user
    ```

#### 3. Returning the Book or the Due Date Passing

**a. Update Asset Availability on Blockchain:**

1. When the user returns the book or the due date passes, the smart contract updates the `isCheckedOut` status to `false`.
    ```rust
    pub fn return_book(env: Env, book_id: Symbol, user: Address) {
        let is_checked_out = env.storage().has(&book_id);

        if !is_checked_out {
            panic!("Book does not exist");
        }

        let checked_out_status: bool = env.storage().get(&book_id).unwrap().unwrap();
        if !checked_out_status {
            panic!("Book is already returned");
        }

        let checked_out_by_key = Symbol::from_str("checkedOutBy");
        let stored_user: Address = env.storage().get(&checked_out_by_key).unwrap().unwrap();
        if stored_user != user {
            panic!("Unauthorized return");
        }

        env.storage().set(&book_id, &false);
        let token_key = Symbol::from_str("checkoutToken");
        env.storage().remove(&token_key);
    }
    ```

**b. Expire Key or Invalidate Token:**

1. The smart contract invalidates the token, making the eBook unreadable to the previous borrower.
    ```rust
    pub fn invalidate_token(env: Env, book_id: Symbol) {
        let return_date_key = Symbol::from_str("returnDate");
        let stored_return_date: Timestamp = env.storage().get(&return_date_key).unwrap().unwrap();
        
        if env.now() > stored_return_date {
            env.storage().set(&book_id, &false);
            let token_key = Symbol::from_str("checkoutToken");
            env.storage().remove(&token_key);
        }
    }
    ```

### Conclusion

This detailed walkthrough outlines the process of adding an eBook to the Stellar blockchain as an asset and the steps involved in the checkout and return processes using Soroban smart contracts. By leveraging blockchain technology, we can ensure that eBooks are securely and efficiently managed, maintaining the principles of the first-sale doctrine in the digital age.
