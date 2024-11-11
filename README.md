# blind-auction-contract
Here's a detailed `README.md` for your Blind Auction smart contract repository:

---

# Blind Auction Smart Contract

This repository contains a Solidity-based Blind Auction contract, where bidders can submit hidden (blinded) bids, and only reveal them after the bidding phase. The contract includes secure handling of bids and deposits, enforces strict timelines for bidding and revealing, and ensures funds are transferred to the winning bidder only after proper verification.

## Features
- **Blind Bidding**: Allows bidders to submit hidden bids with deposits, ensuring bid privacy.
- **Reveal Phase**: Bidders reveal their bids to validate the highest bidder, with the option to mark fake bids.
- **Error Handling**: Enforces strict auction timelines and error handling for a smooth auction experience.
- **Secure Fund Management**: Bidders can withdraw deposits if they do not win, and the highest bid is sent to the beneficiary.
  
## Contract Overview

### State Variables
- **`beneficiary`**: The address that will receive the highest bid amount.
- **`biddingEnd`**: The timestamp indicating the end of the bidding phase.
- **`revealEnd`**: The timestamp indicating the end of the reveal phase.
- **`ended`**: A boolean flag to prevent multiple auction ends.
- **`bids`**: Maps each address to an array of `Bid` structs.
- **`highestBidder` and `highestBid`**: Track the highest bid and its bidder.
- **`pendingReturns`**: Tracks refundable deposits for non-winning bidders.

### Structs
- **`Bid`**: Contains a blinded bid (hashed value) and the deposit associated with the bid.

### Errors
- **`TooEarly(uint time)`**: Triggered when an action is attempted before the designated time.
- **`TooLate(uint time)`**: Triggered when an action is attempted after the designated time.
- **`AuctionEndAlreadyCalled()`**: Triggered when `auctionEnd` is called more than once.

## Functions

### `constructor`
Initializes the auction:
- **Parameters**:
  - `biddingTime`: Duration of the bidding phase.
  - `revealTime`: Duration of the reveal phase.
  - `beneficiaryAddress`: Address to receive the auction funds.

### `bid(bytes32 blindedBid)`
Allows participants to place a blinded bid:
- **Parameters**:
  - `blindedBid`: A hashed value of the actual bid amount, fake status, and a secret.
- **Requirements**:
  - Must be sent with a deposit.
  - Can only be called before `biddingEnd`.

### `reveal(uint[] calldata values, bool[] calldata fakes, bytes32[] calldata secrets)`
Enables bidders to reveal their actual bids and validate deposits:
- **Parameters**:
  - `values`: Array of actual bid amounts.
  - `fakes`: Array indicating if the bid is real or fake.
  - `secrets`: Array of secrets used for hashing.
- **Requirements**:
  - Must match the lengths of each array.
  - Can only be called after `biddingEnd` and before `revealEnd`.

### `withdraw()`
Allows non-winning bidders to withdraw their deposits.

### `auctionEnd()`
Ends the auction and transfers the highest bid amount to the beneficiary:
- **Requirements**:
  - Can only be called after `revealEnd`.
  - Can only be called once.

### `placeBid(address bidder, uint value)`
An internal function to update the highest bid.

## Usage

### Deploying the Contract
1. Deploy the contract with desired `biddingTime` and `revealTime`.
2. Provide the `beneficiaryAddress` that will receive the auction funds.

### Bidding Phase
1. Create a blinded bid by hashing `(value, fake, secret)` using `keccak256`.
2. Call `bid()` with the blinded bid hash and a deposit.

### Reveal Phase
1. Reveal the bids by calling `reveal()` with the original values, fake flags, and secrets.
2. Valid bids are compared, and the highest bid is updated if applicable.

### Ending the Auction
1. After the reveal phase ends, call `auctionEnd()` to finalize and transfer the funds to the beneficiary.

### Withdrawal of Deposits
Non-winning bidders can call `withdraw()` to get their deposits back.

## Example Code

```javascript
const auction = await BlindAuction.deployed();
const value = web3.utils.toWei("1", "ether");
const fake = false;
const secret = web3.utils.sha3("randomSecret");

// Blinded bid: hash of (value, fake, secret)
const blindedBid = web3.utils.soliditySha3(value, fake, secret);

// Placing a bid
await auction.bid(blindedBid, { value: value, from: accounts[0] });

// Reveal bid
await auction.reveal([value], [fake], [secret], { from: accounts[0] });

// End auction after reveal phase
await auction.auctionEnd();
```

## Events
- **`AuctionEnded(address winner, uint highestBid)`**: Triggered when the auction ends and announces the winner.

## Security Considerations
- Ensure to keep secrets confidential to prevent revealing bid amounts prematurely.
- Bidders should reveal bids within the reveal period, or their deposits may be forfeited.
- Only the beneficiary can receive the highest bid amount after the auction.

## License
This contract is open-sourced under the GPL-3.0 License.

--- 

This `README.md` provides clear setup instructions, usage examples, and security notes for developers and users to understand and interact with the Blind Auction smart contract effectively.
