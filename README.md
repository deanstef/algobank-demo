# AlgoBank
This repo introduces _AlgoBank_, the _very_ first [ARC4](https://github.com/algorandfoundation/ARCs/blob/main/ARCs/arc-0004.md) compliant Algorand smart contract written in [PyTeal](https://github.com/algorand/pyteal). __ARC4__ introduces the __Algorand ABI__: a specification that defines how to call Algorand smart contract methods, and the encoding/decoding of the methods parameters and return values.

_AlgoBank_ is an Algorand application that acts as an _escrow_ account for users' funds. Users can call the _AlgoBank_ methods to `deposit` funds and `optin` to the application, get their available `balance` deposited, and finally `withdraw` their funds for themselves or for another user.

This application illustrates how to use the new PyTeal features for the ABI support, such as:

- use of brand new `abi` types;
- usage of the `Router` object taking care of methods and bare calls invocations;
- generation of the contract description JSON file.

This tutorial will guide you toward the deployment and testing of the _AlgoBank_ smart contract, will help you understanding how the PyTeal Algorand ABI works, and finally how to call ABI methods from the CLI.

**⚠️ Disclamer: This code is for demonstration only and has not been audited!**

[AlgoBank is also deployed on testnet!](https://testnet.algoexplorer.io/application/114521775) Check it out on __APPID=114521775__

## How to use it?

### Requirements

1. Algorand [Sandbox](https://github.com/algorand/sandbox) in `dev` mode!
2. [Pipenv](https://pipenv.pypa.io/en/latest/) dependencies management and virtual envs tool.

## Installation

The `Pipfile` in this repo includes all the dependencies to startup a Python `venv` for _AlgoBank_. Using `pipenv` command run:

    pipenv install -d

Enter your virtual env

    pipenv shell

### Deployment

The _AlgoBank_ smart contract must be deployed on your local Algorand private testnet. To do so, first generate the `TEAL` source codes for both the AlgoBank `approval` and `clear state` programs. To learn more about deploying Algorand smart contract written in PyTeal refer the [Algorand developer portal](https://developer.algorand.org/docs/get-details/dapps/pyteal/).

Both files must be copied into the Sandbox using the `copyTo` command:

     ./sandbox copyTo algobank_approval.teal
     ./sandbox copyTo algobank_clear_state.teal

The _AlgoBank_ can now be locally deployed using the `goal` command `app create`:

    ./sandbox goal app create --creator $ACCT1 --approval-prog algobank_approval.teal --clear-prog algobank_clear_state.teal --global-byteslices 0 --global-ints 1 --local-byteslices 0 --local-ints 1 --note "Hello AlgoBank!"

> This tutorial assumes that the Algorand accounts available with the Sandbbox have been aliased as `ACCT1` and `ACCT2`, as env variables, as well as the application id `APPID` and application address `APPACCT`. To obtain the app account just run `./sandbox goal app info --app-id $APPID`

Note that _AlgoBank_'s `StateSchema` only requires one `Global` and one `Local` Integers.

Fund AlgoBank with a minimum balance of 0.1 `ALGO`:

    ./sandbox goal clerk send --from $ACCT1 --to $APPACCT --amount 100000`

### Usage

#### Deposit funds

To deposit funds a user must call the `deposit` ABI method. This method takes two arguments: a `payment` transaction and an Algorand `account`. First, the `payment` transaction must be generated, then use the `method` CLI. Foe example run:

    ./sandbox goal clerk send --from $ACCT2 --to $APPACCT --amount 1000000 -o payment.txn
    ./sandbox goal app method --app-id $APPID --method "deposit(pay,account)void" --arg payment.txn --arg $ACCT2 --from $ACCT2 --on-completion OptIn

The method signature is passed as method parameter along with the method arguments and the `OptIn` on completition. The example above deposits 1 `ALGO` into the AlgoBank account of `ACCT2`.

#### Check the balance

After depositing funds, a user might be interested in checking his available balance. _AlgoBank_ exposes the `getBalance` method for doing that. This method returns the balance value, which will be decoded by the ABI (clients do not care about encoding/decoding operations with ABI):

    ./sandbox goal app method --app-id $APPID --method "getBalance(account)uint64" --arg $ACCT2 --from $ACCT2

#### Withdraw funds

Withdrawals can be accomplished by calling the `withdraw` method of the application. For example:

    ./sandbox goal app method --app-id $APPID --method "withdraw(uint64,account)void" --arg 250000 --arg $ACCT2 --from $ACCT2 --fee 2000

A 0.25 `ALGO` has been withdrawn from the AlgoBank account of `ACCT2`, and moved to the `ACCT2`'s wallet.
