# Docs

## Contracts

### Opportunities

- Voting (for, against, abstain)
  - Advanced
    - User can write a contract and it will be executed
  - Simple
    - Nothing will be enforced
      - People just vote in favor of staircase repair
  - Voting templates
    - Voting for grant payment
      - Who to pay and how much (in native currency)
      - Who to pay and how much in ERC20 token
    - Additional Mint tokens
- Treasury
  - A contract where tokens can be transferred to be paid out to someone by vote
- TokenDAO
  - Used for voting only, but can be traded as well

### SuperDAO

Since contracts cannot be deployed in stages in Forward Factory, we had to make a contract that will deploy all contracts in one transaction, which turned out to be successful and tested when deploying the template on the marketplace

### MiniDAO

The basis for the DAO, the whole logic of voting and governance.

### TimeLock

Contract to create a delay in the execution of the proposal.

### TokenDAO

Voting token contract, `mint` is only available for `owner` which is equal to TimeLock. That is, additional issue is available only after voting.

### Treasury

Contract for fundraising to the DAO, this can be used to disburse money from the fund to specific addresses. Disbursement functions are only available after voting.

## Functions

### `propose`

Create a proposal to be voted on.

- `targets`- array of addresses to be accessed
- `values` - send native tokens
- `calldatas` - array of selectors of functions + parameters (calldatas)
- `description` - proposal description

#### Advanced

The proposer himself creates the contract, which will then be enforced

#### Simple

The `propose` parameters are set to zeros, the result will be a vote, but nothing will be executed

```ts
const targets: string[] = ["0x0000000000000000000000000000000000000000"];
const values: number[] = [0];
const calldatas: string[] = ["0x00"];
```

#### Templates

- Additional TokenDAO minting

```ts
const target = token.target;
const calldata = (await token.mint.populateTransaction(acc5, 10000)).data;
const description = "Mint 10000 tokens";
await miniDAO.propose([target], [0], [calldata], description);
```

- Pay the grant to the address in native tokens

```ts
let ABI = ["function releaseNativeToken(address to, uint256 amount)"];
const iface = new hre.ethers.Interface(ABI);

const target = treasury.target;
const calldata = iface.encodeFunctionData("releaseNativeToken", [
  acc6.address,
  hre.ethers.parseEther("1.1"),
]);
const description = "Pay out a grant to the first team";
await miniDAO.propose([target], [0], [calldata], description);
```

- Pay the grant to the address in ERC20 tokens

```ts
let ABI = [
  "function releaseERC20Token(address to, uint256 amount, address token)",
];
const iface = new hre.ethers.Interface(ABI);
const calldata = iface.encodeFunctionData("releaseERC20Token", [
  acc6.address,
  200000,
  erc20.target,
]);
const target = treasury.target;
const description = "Pay out a grant to the second team";
await miniDAO.propose([target], [0], [calldata], description);
```

- Pay the grant to the address in ERC721 token

```ts
let ABI = [
  "function releaseERC721Token(address to, uint256 id, address token)",
];
const iface = new hre.ethers.Interface(ABI);
const calldata = iface.encodeFunctionData("releaseERC721Token", [
  acc6.address,
  0,
  erc721.target,
]);

// const calldata = (
//   await treasury.releaseERC721Token.populateTransaction(acc6, 0, erc721)
// ).data;

const target = treasury.target;
const description = "Pay out a grant to the third team";
await miniDAO.propose([target], [0], [calldata], description);
```

### `castVote`

Vote for/against

- `proposalId`
- `support` - [ Against, For, Abstain ] = [0, 1, 2]

### `proposalVotes`

Get voting statistics for the proposal

- [ Against, For, Abstain ]
