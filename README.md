<p align="center">
  <img src="https://calyptus.co/wp-content/uploads/logo.svg" width="300" title="Calyptus" >
</p>
<br />

# Calyptus Smart Contract Security Challenge



Time to get your hands dirty!

In this section of the course, you get to use the smart contracts related skills you have acquired so far. This section has 10 challenges, and the difficulty of the challenges increases as you go further.

The game has two characters, Alice and Bob. You help Bob who is trying to give Alice a hard time. Some challenges may time to time introduce other characters as well.

Some challenges require Bob to steal the funds from Alice's smart contracts, some want Bob to halt Alice's smart-contracts using a DOS and some want Bob to blow Alice's mind.

Are you ready to play the game?

## System Requirements

To use this repo you should have the following programmes installed in your machine:

- Node ^18.12.1 [installation guide](https://nodejs.dev/en/learn/how-to-install-nodejs/)
- Yarn Package Manager [installation guide](https://classic.yarnpkg.com/lang/en/docs/install)

## How to play

### Clone this repo, cd into it and install the dependencies by pasting the following code into your terminal:

```bash
git clone https://github.com/Calyptus-Learn/SC-Security-Challenges.git && cd SC-Security-Challenges && yarn install
```

### Start Solving the challenges

The `contracts` folder consists of the smart-contracts mentioned in each challenge. `test` folder consists of the setup scripts of each challenge. Pass the test to win the challenge.

Some challenges would need you to deploy your own smart-contracts, write those in the `contracts/BobsContracts` folder.

Use ethers.js to code the steps of your hack into the `Exploit` section of the test script. Use Bob's account to perform all the exploits by using [.connect(bob)](https://docs.ethers.io/v5/single-page/#/v5/api/contract/contract/-%23-Contract-connect) notation from ethers.js

The default "Exploit" section of all the test files look like this:

```js
it("Exploit", async function () {
  /** CODE YOUR EXPLOIT HERE  */
});
```

After writing your exploit, run this script from your terminal:

```bash
npx hardhat test <PATH_TO_THE_TEST>
```

---

## Challenge 1

### No Privacy (Vulnerability: Accessing Private Variables)

---

Alice has deployed a secret Lock on blockchain that opens with a password. Help Bob find out the password and unlock the lock to win this challenge.

**Check out the [No Privacy](contracts/NoPrivacy/AlicesLock.sol) smart-contract and find a way to unlock it.**

**Pass this [Test](test/no-privacy.js) to win the challenge.**

### Solution Notes

In Ethereum, smart contract data is stored in a section of the blockchain called the contract's storage. This storage is divided into slots, with each slot capable of storing a 32 byte word. Smart contract variables are assigned slots in the order they are declared in the contract, starting from 0. This contract storage is publicly accessible to anyone that knows how to read it.

The `getStorageAt` function provided by ethers.js allows you to read the raw data in a specific storage slot of a given Ethereum address. In your smart contract, the `password` variable is stored at slot 1 (as the `locked` boolean variable is at slot 0), so calling `getStorageAt(lock.address, 1)` retrieves the bytes32 password value from the smart contract's storage.

Therefore, the reason why the `ethers.provider.getStorageAt` function is used in this scenario to "hack" the contract is because the `password` variable is not really private. Despite the password being marked as a private variable, it's important to know that 'private' in Solidity does not mean 'hidden'. It only means that it can't be accessed directly by other contracts, but its value is still stored openly on the blockchain.

In the context of the test, the `getStorageAt` function is used to retrieve the `password` from the contract's storage and then call the `unlock` function with that password, which unlocks the lock and allows the test to pass. This highlights an important concept in smart contract development: sensitive data, like passwords or private keys, should never be stored directly on-chain, because they can be easily retrieved using functions like `getStorageAt`.

---

## Challenge 2

### Head or Tail (Vulnerablilty: Randomness through global variable)

---

Alice and Bob are flipping a coin to decide who is a better smart contract programer.

Help Bob win the coin flip 5 times in a row to win the challenge.

**Check out the [Head or Tail](contracts/HeadOrTale/HeadOrTail.sol) smart-contract and find a way to hack it.**

**Pass this [Test](test/head-or-tail.js) to win the challenge.**

### Solution Notes

The HeadOrTail contract is a simple coin flip game where the result of the flip is pseudo-randomly generated by taking the hash of the previous block (blockhash(block.number - 1)) and dividing it by a certain FACTOR. If the result is 1, it means "heads" (represented as true) and if it's not 1, it means "tails" (represented as false). If the player's guess is correct, they win and the 'wins' counter increments by 1.

The vulnerability here lies in the way the contract generates randomness. In Ethereum, all data is transparent, including the block hash. It means that one can predict the result of the coin flip by calculating it the same way the contract does.

Bob exploits this vulnerability by creating another contract (BobsGuess) that calculates the result of the coin flip before actually flipping the coin in the HeadOrTail contract. Specifically, BobsGuess takes the address of the HeadOrTail contract as a parameter, calculates the expected result of the coin flip (by using the same logic as the flip function), and then calls the flip function with the expected result.

In the test, an instance of BobsGuess is created and the cheat function is called 5 times in a row, each time predicting the result of the coin flip and then flipping the coin with the predicted result, which results in Bob winning the coin flip 5 times in a row.

This challenge demonstrates the risks of generating randomness on-chain and relying on block variables for randomness. Even though block variables like block.timestamp or block.number seem unpredictable, they can be manipulated or predicted by miners or, in some cases, by other smart contracts. Therefore, it's generally considered unsafe to use block variables to generate random numbers in Ethereum smart contracts, especially when the result of the randomness could lead to a significant gain or loss for any party involved.

---

## Challenge 3

### Mount Calyptus (Vulnerability: Denial of Service due to push pattern)

---

Everyone wants to be at the top of Mount Calyptus, but there's space for only one. As they say, everything can be bought with money, so can be the spot at the summit. Whoever sends the Mount Calyptus Smart Contract an amount of ether that is larger than the current bribe replaces the previous climber. On such an event, the replaced climber gets paid the new bribe, making a bit of ether in the process!

Alice wants to be at the top at all cost! Alice reclaims the top spot as soon as anyone claims it sending equal bribe.

Help Bob stop Alice from reclaiming the atTheTop position.

**Check out the [Calyptus Hill](contracts/CalyptusHill/CalyptusHill.sol) smart-contract and find a way to hack it.**

**Pass this [Test](test/calyptus-hill.js) to win the challenge.**

### Solution Notes

The challenge deals with a vulnerability in Ethereum smart contracts known as a Denial of Service (DoS) attack due to an unhandled exception. A contract can become vulnerable to this attack when it relies on transfer or send function to forward funds to another address, without handling the possibility of these operations failing.

The `CalyptusHill` contract allows anyone to become `atTheTop` if they send an amount of Ether that is equal to or larger than the current `bribe`. The sent bribe is then forwarded to the current `atTheTop` address. Initially, Alice is atTheTop and whenever someone sends an equal or larger bribe to take her spot, she quickly sends the same amount to reclaim her position.

The vulnerability arises in the `receive` function of the `CalyptusHill` contract, where the contract tries to forward the incoming Ether to the current `atTheTop` address using the `transfer` function. If the `transfer` call fails for any reason (e.g., the receiving contract runs out of gas or throws an exception), it will cause the entire `receive` function to revert, effectively preventing the execution of any code that follows it.

Bob's goal is to take the `atTheTop` position and prevent Alice from reclaiming it. To achieve this, Bob deploys the `HillAttack` contract. This contract doesn't have a payable fallback function, which means it will automatically throw an exception if it receives Ether via a `transfer` or `send` call.

When Bob calls the `attack` function of `HillAttack` contract, it sends Ether to the `CalyptusHill` contract, which tries to forward this incoming Ether to the current `atTheTop` address (Alice initially). After the first attack transaction, the `atTheTop` position has been claimed by Bob's `HillAttack` contract. Now, whenever CalyptusHill tries to send Ether to `HillAttack` (i.e., when Alice attempts to reclaim her position), it will fail and revert due to the lack of a payable fallback function in `HillAttack`. This prevents Alice from reclaiming the `atTheTop` position, effectively locking it to Bob's `HillAttack` contract.

This example highlights a key principle in Ethereum smart contract development: be careful when using external calls, especially if they involve transferring Ether. External calls can introduce potential vulnerabilities, especially when not correctly handled or placed at the end of your functions.

---

## Challenge 4

### Do Not Trust (Vulnerability: Insecure External Call)

---

Alice has deployed a `Do Trust Lender` pool that offers free flash-loans to everyone!

Awesome right?

The pool has 1 million Calyptus Tokens (CPT) in balance. Complete the challenge by making Bob steal all the CPTs from the pool and send them into the his account.

**Check out the [Do Trust Lender Pool](contracts/DoNotTrust/DoTrustLender.sol) smart-contract and find a way to hack it.**

**Pass this [Test](test/do-not-trust.js) to win the challenge.**

### Solution Notes


The "Do Not Trust" challenge revolves around a vulnerability commonly referred to as an "Insecure External Call". This particular challenge centers around a smart contract that provides a feature of a flash loan, where a user can borrow tokens under the condition that they are returned within the same transaction.

Here is how the vulnerability in the DoTrustLender smart contract is exploited:

* The `flashLoan` function of the `DoTrustLender` contract allows any contract to borrow tokens and call a function on any contract with provided data. This is where the vulnerability lies.
* The `DoTrustLender` contract checks whether the loan was paid back by comparing the balance of the tokens before and after the call to the target function. If the balance of tokens in the contract after the call is less than before, the function reverts.
* The `DoNotTrustBob` contract's `attack` function calls `flashLoan` on the `DoTrustLender` contract, specifying that 0 tokens should be transferred to it, and that it should call the `approve` function on the token contract to allow itself to spend the lender contract's tokens.
* This works because the `DoTrustLender` contract first transfers the tokens and then makes the external call. This means that `approve` is called after the tokens are already transferred, and there's nothing stopping the `DoNotTrustBob` contract from spending them.
* After the `flashLoan` call, the `DoNotTrustBob` contract transfers all of the `DoTrustLender` contract's tokens to itself, effectively draining all tokens from the lender contract.

The crux of this exploit is the dangerous practice of allowing untrusted contracts to execute arbitrary function calls. This allows the attacking contract to manipulate the state of the lending contract in a way that wasn't intended, leading to a loss of funds.

A good rule of thumb when designing secure smart contracts is to always be aware of re-entrancy attacks and state changes during external calls. It's also a good practice to use the Checks-Effects-Interactions pattern, which recommends handling any state changes before making external calls.

---

## Challenge 5

### Re-enter (Vulnerability: Reentrancy)

---

Alice has deployed a simple lending pool that allows its users to deposit ETH, and withdraw it at any point in time.

This simple lending pool already has 1000 ETH in balance, and is offering free flash loans using the deposited ETH.

Help Bob steal all the ETH from Alice's lending pool.

**Check out the [ReenterPool](contracts/Reenter/Reenter.sol) smart-contract and find a way to hack it.**

**Pass this [Test](test/reenter.js) to win the challenge.**

### Solution Notes

This challenge involves exploiting a reentrancy vulnerability in the lending pool contract named `ReenterPool` deployed by Alice. This contract provides a feature to deposit Ether and a feature to withdraw it at any time. It also provides a flash loan feature that allows anyone to borrow any amount of Ether as long as they return it within the same transaction.

The attacker contract `ReentersBob` is designed to steal all the Ether from the `ReenterPool` lending pool. Bob deploys the `ReentersBob` contract and calls the `attack` function, which initiates a flash loan for the entire balance of the `ReenterPool` contract.

The `flashLoan` function of the `ReenterPool` contract expects a specific function named `execute` to exist in the borrower's contract. The `execute` function is called immediately after the loan is disbursed. In `ReentersBob` contract, the `execute` function is designed to reenter the `ReenterPool` contract by calling its `deposit` function, depositing the loaned amount back into the pool.

Then, the `attack` function of `ReentersBob` calls the `withdraw` function of the `ReenterPool` contract. Since `ReentersBob` had previously deposited the loaned amount, it can now withdraw the entire balance of the `ReenterPool`, effectively stealing all the Ether from the pool.

The success of this hack is confirmed by checking the final balance of the `ReenterPool` and Bob. If the `ReenterPool` is empty and Bob's balance has increased, the attack was successful.

This hack exploits a reentrancy vulnerability because `ReentersBob` was able to call back into the `ReenterPool` contract before the `flashLoan` operation had completed. This is a reminder that smart contracts must carefully manage their state and consider the potential for reentrant calls when designing and implementing contract functions.