## The First-Sale Doctrine: A Historical Perspective and Its Modern Relevance

### Introduction

The first-sale doctrine has been a cornerstone of U.S. copyright law, balancing the interests of copyright holders and the public for over a century. It has facilitated the resale, rental, and lending of copyrighted works, contributing significantly to public access to literature, music, and other forms of intellectual property. However, the advent of digital technology and the internet has posed new challenges to this doctrine. This article explores the history and importance of the first-sale doctrine and proposes how blockchain technology, specifically through our eBook checkout system, can help maintain existing market dynamics in the digital era.

### History of the First-Sale Doctrine

The first-sale doctrine originated in the early 20th century with the Supreme Court case **Bobbs-Merrill Co. v. Straus** (1908). The court ruled that once a copyrighted item is sold, the copyright owner no longer controls its distribution. This doctrine was codified in the Copyright Act of 1976, allowing the lawful owner of a copy of a work to sell, lend, or otherwise dispose of that copy without permission from the copyright holder.

### Importance of the First-Sale Doctrine

#### Enhancing Accessibility and Affordability

The first-sale doctrine has played a crucial role in enhancing the accessibility and affordability of copyrighted works. It enabled the establishment of libraries, second-hand bookstores, and rental services, allowing people to access works without purchasing them new. This has been particularly beneficial for educational institutions and low-income individuals.

#### Fostering a Secondary Market

The doctrine has also fostered a robust secondary market for books, music, movies, and other media. This market has provided additional revenue streams for sellers and wider choices for consumers. It has allowed for the circulation of out-of-print works, preserving cultural and intellectual heritage.

#### Balancing Interests

By limiting the control of copyright holders over the distribution of individual copies after the initial sale, the first-sale doctrine has balanced the interests of creators and the public. It has ensured that once a work is legally purchased, the purchaser has the right to use and distribute it as they see fit, within the confines of the law.

### Challenges in the Digital Era

With the rise of digital technology, the first-sale doctrine faces significant challenges. Digital works, unlike physical copies, can be copied and distributed with ease, leading to potential widespread unauthorized dissemination. Technological protection measures and digital rights management (DRM) systems have been employed to control access and distribution, but these measures often limit legitimate secondary uses of digital works.

#### Impact of Digital Networks

Digital networks enable the distribution of works without transferring ownership of a physical copy, complicating the application of the first-sale doctrine. The ease of copying digital works has led copyright holders to retain more control over distribution, often at the expense of consumer rights and secondary markets.

#### Legal and Technological Measures

The Digital Millennium Copyright Act (DMCA) of 1998 further complicated the landscape by providing legal support for technological protection measures. While these measures help prevent unauthorized copying, they also restrict lawful secondary uses and diminish the applicability of the first-sale doctrine in the digital realm.

### Blockchain eBook Checkout Proposal: A Modern Solution

To address these challenges, our blockchain eBook checkout proposal leverages the security and transparency of blockchain technology, specifically using the Stellar network and Soroban smart contracts. This system aims to uphold the principles of the first-sale doctrine in the digital age.

#### Creating and Issuing Digital Assets

We propose creating a digital asset for each eBook on the Stellar network, with custom properties to manage its lending status. This ensures that each digital copy can be uniquely identified and tracked.

#### Smart Contracts for Management

Soroban smart contracts manage the checkout and return processes, generating a unique token for each loan. This token is used to encrypt the eBook, ensuring that only the borrower can access it. Upon return, the token is invalidated, and the eBook is no longer accessible to the borrower.

```rust
#![no_std]

use soroban_sdk::{contractimpl, Env, Symbol, Address, BytesN, Timestamp};

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

#### Ensuring Access and Revocation

This system ensures that eBooks can only be read by one person at a time and that access is revoked when the book is returned or the lending period expires. By leveraging blockchain technology, we can maintain the principles of the first-sale doctrine in the digital realm.

### Conclusion

The first-sale doctrine has been instrumental in balancing the interests of copyright holders and the public, fostering a secondary market, and enhancing access to intellectual property. However, the digital age poses significant challenges to this doctrine. Our blockchain eBook checkout proposal offers a modern solution, using Stellar and Soroban smart contracts to ensure secure and controlled lending of digital works. By adapting the first-sale doctrine to the digital era, we can preserve its benefits and continue to promote access to knowledge and culture.

### Future Work

Further work could involve:
- Integrating the system with existing library management software.
- Developing user-friendly interfaces for library staff and patrons.
- Implementing additional features such as notifications for overdue books and automated extensions.

This proposal aims to leverage cutting-edge blockchain technology to modernize library operations and improve the accessibility and security of digital resources. We look forward to the opportunity to implement and demonstrate this innovative solution.
