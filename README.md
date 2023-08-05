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

By setting msg.value so that CALLVALUE pushes the right value onto the stack, we can control where the JUMP instruction goes, landing us atthe JUMPDEST we want.
This lets us proceed to STOP and halt execution successfully.
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

[Puzzle 4](https://www.evm.codes/playground?callValue=6&unit=Wei&callData=&codeType=Bytecode&code=%2734381856FDFDFDFDFDFD5B00%27_&fork=shanghai) :

```
The goal is to make the CODESIZE xor CALLVALUE equal 0xa.

0xa is the location of the JUMPDEST we want to jump to.

CODESIZE is fixed at 0xc (12 in decimal).

We need to find a CALLVALUE that when XOR'ed with 0xc gives 0xa.

Converting 0xc to binary is 1100 (in 4 bits).

0xa in binary is 1010.

Using XOR, we need a CALLVALUE where:

1100 (CODESIZE)
XOR
0110 (CALLVALUE)
= 1010 (0xa)

Looking bit by bit, 0110 in binary XOR'd with 1100 gives 1010.

0110 in hex is 0x6.

So we need CALLVALUE of 0x6.

This will make CODESIZE xor CALLVALUE equal 0xa, causing the JUMP to go to JUMPDEST at 0xa.

In summary:

We calculate the CALLVALUE of 0x6 that will make the XOR equation work out to jump to 0xa.
```

[Puzzle 5](https://www.evm.codes/playground?callValue=16&unit=Wei&callData=&codeType=Bytecode&code=%2734800261010014600C57FDFD5B00FDFD%27_&fork=shanghai) :

```
The goal is to make the JUMPI jump to the JUMPDEST at 0x0c.

JUMPI will jump if the stack value below it is 1.

Before JUMPI:

- CALLVALUE (the amount sent) is duplicated
- The duplicates are multiplied, squaring CALLVALUE
- This result is compared to 0x0100 (256 decimal)
- 1 is pushed if equal, 0 otherwise

So we need to send a CALLVALUE that when squared equals 256.
This will make the comparison push 1 onto the stack.

The square of 16 is 256.

So sending CALLVALUE=16 will:

- Duplicate to 16, 16
- Square to 256
- Equals 256 so push 1

Then JUMPI sees a 1 below it on the stack so it jumps to 0x0c.

In summary:

We calculate CALLVALUE=16 to make the math work out so JUMPI jumps to the target JUMPDEST.
```

[Puzzle 6](https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x000000000000000000000000000000000000000000000000000000000000000a&codeType=Bytecode&code=%2760003556FDFDFDFDFDFD5B00%27_&fork=shanghai) :

```
The goal here is to make CALLDATALOAD(0) return 0x0a.

CALLDATALOAD loads 32 bytes of data from calldata into memory.

The 0 offset means it will load the first 32 bytes.

This loaded value needs to be 0x0a so that the JUMP lands on the JUMPDEST at 0x0a.

To load 0x0a, we have to pass 32 bytes of calldata where the first byte is 0x0a.

0x0a is only 1 byte long. So we pad it to 32 bytes like this:

0x000000000000000000000000000000000000000000000000000000000000000a

This 32-byte sequence has 0x0a as the first byte.

When CALLDATALOAD(0) loads it, only the first byte matters.

It will return 0x0a which makes JUMP go where we want.

In summary:

We pad the 1-byte 0x0a to 32 bytes and pass that as calldata so CALLDATALOAD gives us 0x0a.
```

[Puzzle 7](https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x608060005260015ff3&codeType=Bytecode&code=%2736600080373660006000F03B600114601357FD5B00%27_&fork=shanghai) :

```
The goal is to make the JUMPI jump to the JUMPDEST at location 0xd.

JUMPI jumps if the top stack value is 1.

We need to make the EQ comparison before it push 1.

The EQ compares EXTCODESIZE from the CREATE result of 1.

So we need to create a CALLDATA of a contract with EXTCODESIZE of 1 byte.
```

[Puzzle 8](https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x60fd6000526001601ff3&codeType=Bytecode&code=%2736600080373660006000F03B600114601357FD5B00%27_&fork=shanghai) :

```
The goal is to deploy a contract using CREATE that reverts when called.

This will make the CALL return 0.

Comparing 0 to 0 will be true, pushing 1 to stack.

The JUMP will then jump over the reverts.

To create a reverting contract:

We need calldata that has just a REVERT opcode.
0xfd is the REVERT opcode.
We'll pad it to 3 bytes to make a valid contract.
So the calldata can just be:

0x60fd6000526001601ff3

This contains:

0xfd - REVERT opcode
Padding to 3 bytes
When CREATE deploys this:

Calling the contract will just REVERT
CALL returns 0
0 compared to 0 is true
JUMP succeeds
So deploying a simple REVERT contract solves this puzzle.
```

[Puzzle 9](https://www.evm.codes/playground?callValue=2&unit=Wei&callData=0x01020304&codeType=Bytecode&code=%2736600310600957FDFD5B343602600814601457FD5B00%27_&fork=shanghai) :

```
The goal is to make CALDATASIZE * CALLVALUE = 8

CALDATASIZE needs to be > 3 based on the first check.

The smallest size > 3 is 4 bytes.

So we can pass any 4 bytes of data, like 0x01020304.

Now CALDATASIZE will be 4.

To get 8 when multiplied by CALDATASIZE, CALLVALUE needs to be 2.

Because:

CALDATASIZE = 4
CALLVALUE = 2

4 * 2 = 8

This will make the equality comparison true.

So in summary:

- Pass 4 bytes as calldata to make SIZE = 4
- Pass CALLVALUE of 2
- 4 * 2 is 8, so multiplication passes the check
```

[Puzzle 10](https://www.evm.codes/playground?callValue=15&unit=Wei&callData=0x010203&codeType=Bytecode&code=%2738349011600857FD5B3661000390061534600A0157FDFDFDFD5B00%27_&fork=shanghai) :

```
The goal is to:

1. Make CALDATASIZE % 3 == 0
2. Make CALLVALUE + 0xa == 0x19

To make CALDATASIZE % 3 == 0:

- CALDATASIZE just needs to be divisible by 3
- Sending any 3 bytes of data works, e.g. 0x010203

Now CALDATASIZE is 3.

For CALLVALUE:

- Needs to be < CODESIZE which is 0x1b = 27
- Added to 0xa needs to equal 0x19
- 0x19 - 0xa is 0xf = 15
- So CALLVALUE of 15 works

15 < 27, so CALLVALUE is valid.

15 + 0xa = 0x19, so JUMP destination reached.

So in summary, calldata of 3 bytes and CALLVALUE of 15 solves it.
```
