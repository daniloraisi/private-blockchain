# Private Blockchain Application

> This project is part of the Udacity Blockchain Developer Nanodegree Program

This project allows me to demonstrate the fundamental concepts of a Blockchain platform.

Concepts like:

- Block
- Blockchain
- Wallet
- Blockchain Identity
- Proof of Existance

I created this project with a boilerplate code containing a REST API to expose some of the functionalities needed for my private blockchain to work.

## What problem this private Blockchain application solves?

An employer is trying to test a concept on how his company can use a Blockchain application.

He is an astronomy fan and spends most of his free time searching for stars in the sky. That's why he would like to create a test application that will allow him to register stars, and also some others of his friends can register stars too but make sure the application knows who owns each star.

### What is the process described by the employer to use to develop the application?

1.  The application will create a Genesis Block when we run it for the first time.

2.  The user will request the application to send a message to be signed using a Wallet and, in this way, verify the ownership over the wallet address. The message format will be:

    `<WALLET_ADRESS>:${new Date().getTime().toString().slice(0,-3)}:starRegistry`

3.  Once the user has the message, he can use a Wallet to sign this message.

4.  The user will try to submit the Star object, for that it will send: `wallet address`, `message`, `signature` and the `star` object with the star information.
    The Start object needs to follow this format:

        ```json
        "star": {
            "dec": "68° 52' 56.9",
            "ra": "16h 29m 1.0s",
            "story": "Testing the story 4"
        }
        ```

5.  The application will verify if the time elapsed from the request ownership (the time contained in the message) and when you submit the star is less than 5 minutes.

6.  If everything is okay, the star information will be stored in the Block and added to the `chain`.

7.  The application will allow us to retrieve the Star objects that belong to an owner (wallet address).

## What tools or technologies are involved in creating this application?

- This application was created using Node.js and JavaScript programming language. The architecture uses ES6 classes because they help us organize the code and facilitate the maintenance of the code.

- The company suggested using Visual Studio Code as an IDE to write the code because it helps debug the code easily (but you can choose the code editor you feel comfortable with).

- Some of the libraries and npm modules used:
  - "bitcoinjs-lib": "^5.2.0",
  - "bitcoinjs-message": "^2.2.0",
  - "body-parser": "^1.19.0",
  - "crypto-js": "^4.1.1",
  - "express": "^4.17.1",
  - "hex2ascii": "0.0.3",
  - "morgan": "^1.10.0"

Libraries walkthrough:

1. `bitcoinjs-lib` and `bitcoinjs-message`: Those libraries help us verify the wallet address ownership. We use it to verify the signature.

2. `express`: The REST API created for this project uses the Express.js framework.

3. `body-parser`: This library acts as a middleware module for Express and helps us to read the JSON data submitted in a POST request.

4. `crypto-js`: This module contains some essential cryptographic methods and helps us to create the block hash.

5. `hex2ascii`: This library helps us to **decode** the data saved in the body of a Block.

## Understanding the code

The application has a simple architecture. It includes a REST API application to expose the Blockchain application methods to the client applications or users.

1. `app.js` file: It contains the configuration and initialization of the REST API. I did not change the boilerplate's original code because it is already tested and works as expected.

2. `BlockchainController.js` file: It contains the routes of the REST API. Those are the methods that expose the URLs we need to call when requesting the application.

3. `src` folder: Here, we have the main two classes needed to create our Blockchain application, a `block.js` file and a `blockchain.js` file that contains the `Block` and `BlockChain` classes.

## How was the application implemented to satisfy my employer requirements?

1.  `block.js` file. In the `Block` class, we implemented these methods:

    `validate()`: The `validate()` method validates if the Block has been tampered or not.

    Been tampered means that someone from outside the application tried to change values in the block data. As a consequence, the hash of the Block should be different.

        Steps:
        1. It returns a new promise to allow the method to be called asynchronous.
        2. We save in an auxiliary variable the current hash of the Block
        3. We recalculate the hash of the entire Block (using SHA256 from crypto-js library)
        4. Then, compare if the auxiliary hash value is different from the calculated one
        5. Resolve true or false depending on if it is valid or not

    `getBData()`: Auxiliary Method to return the block body (decoding the data)

        Steps:
        1. We use hex2ascii module to decode the data
        2. Data is a JavaScript object so, we use JSON.parse(string) to get the object
        3. If the Block is not the `genesis block` we resolve with the data

2.  `blockchain.js` file. In the `Blockchain` class, we implemented the methods:

    `_addBlock(block)`: The method returns a Promise that resolves with the Block added or reject if an error happens during the execution.

        Steps:
        1. We check for the height to assign the `previousBlockHash`,
        2. Also, assign the `timestamp` and the correct `height`
        3. We create the `block hash` and push the Block into the chain array

    `requestMessageOwnershipVerification(address)`: This method allows us to request a message that we will use to sign it with a Bitcoin Wallet (Electrum or Bitcoin Core)

        Steps:
        1. This is the first step before submitting the Block.
        2. The method returns a Promise that will resolve with the message to be signed

    `submitStar(address, message, signature, star)`: This method allows users to register a new Block with the star object into the chain. It resolves with the Block added or reject with an error.

        Steps:
        1. Get the time from the message sent as a parameter
        2. Get the current time
        3. Check if the time elapsed is less than 5 minutes
        4. Verify the message with the wallet address and signature
        5. Create the Block and add it to the chain
        6. Resolve with the Block added.

    `getBlockByHash(hash)`: This method returns a Promise that resolves with the Block with the hash passed as a parameter.

        Steps:
        1. Search on the chain array for the Block that has the hash.

    `getStarsByWalletAddress (address)`: Returns a Promise that resolves with an array of Stars objects existing in the chain and that belongs to the owner with the wallet address passed as a parameter.

    `validateChain()`: This method returns a Promise that resolves with the list of errors when validating the chain.

        Steps:
        1. Validates each Block using the `validateBlock` method.
        2. Each Block checks with the previousBlockHash
