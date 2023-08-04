# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

## Solutions explained

[Puzzle 1](https://www.evm.codes/playground?callValue=8&unit=Wei&callData=&codeType=Bytecode&code=%273456FDFDFDFDFDFD5B00%27_&fork=shanghai) :

```
The goal of the code is to reach the STOP opcode and halt execution.

To get to STOP, we first need to jump to the JUMPDEST at program counter location 8.

The JUMP opcode before this uses the value at the top of the stack to decide where to jump to.

The top of the stack comes from the CALLVALUE opcode. This places msg.value (in wei) onto the stack.

So by setting msg.value correctly, we can control the value JUMP will read from the stack.

We need to make JUMP read a value of 8, so it will jump to the JUMPDEST at program counter position 8.

This allows execution to then reach the STOP opcode after it, halting the program as desired.

In summary:

By setting msg.value so that CALLVALUE pushes the right value onto the stack, we can control where the JUMP instruction goes, landing us at the JUMPDEST we want. This lets us proceed to STOP and halt execution successfully.
```

[Puzzle 2](https://www.evm.codes/playground?callValue=4&unit=Wei&callData=&codeType=Bytecode&code=%2734380356FDFD5B00FDFD%27_&fork=shanghai) :

```
The goal is to pass a callvalue that will cause the JUMP to land on the JUMPDEST at location 6.

First, callvalue is pushed to the stack. Then CODESIZE (10) is pushed.

SUB subtracts top - top-1. So we get CODESIZE - callvalue.

This result needs to equal 6 for the JUMP to land at JUMPDEST 6.

Plugging it in:
10 - callvalue = 6
callvalue = 10 - 6 = 4

So we need callvalue=4. This will make the SUB produce 6.

When JUMP reads 6 off the stack, it will jump to JUMPDEST at location 6.

Execution continues past 6, avoiding the REVERTs later.

So the key is passing callvalue=4 to make the math work out so JUMP hits the right JUMPDEST location.
```

[Puzzle 3](https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x00000000&codeType=Bytecode&code=%273656FDFD5B00%27_&fork=shanghai) :

```
The goal here is to make the JUMP opcode jump to the JUMPDEST at location 4.

First, CALLDATASIZE calculates the length of the call data in bytes.

For example, if we pass "0x00" as call data, CALLDATASIZE will push 1 onto the stack.

The JUMP opcode jumps to the location equal to the value at the top of the stack.

So we need to make CALLDATASIZE push a 4 onto the stack.

This will make JUMP go to the JUMPDEST at location 4.

To do this, we can pass a call data value that is 4 bytes long. For example "0x00000000".

CALLDATASIZE will calculate this call data is 4 bytes long, and push 4 onto the stack.

When JUMP executes, it will read 4 from the stack and jump to location 4 - hitting the desired JUMPDEST.

In summary:

We pass 4-byte call data to make CALLDATASIZE push a 4, so JUMP goes where we want.
```
