## Technologies

This chapter will provide an overview of the technologies required to implement the design philosophy of SNRC-20.

### Poseidon HASH

SNRC-20 uses the `Poseidon HASH` algorithm, which is friendly to Zero-Knowledge Proofs, for Off-Chain hash computation.

[Poseidon HASH](https://www.poseidon-hash.info/) is a family of hash functions designed to be very efficient as algebraic circuits. As a ZK-friendly hashing, they can be very useful in ZK-proving systems such as STARKs.

Poseidon is a sponge construction based on the Hades permutation. Starknet’s version of Poseidon is based on a three-element state permutation.

A Poseidon hash of up to 2 elements is defined as follows.

\\[
poseidon_1(x) := \left[\text{hades_permutation}(x,0,1)\right]_0
\\]

\\[
poseidon_2(x,y) := \left[\text{hades_permutation}(x,y,2)\right]_0
\\]

Where \\( [\cdot]_j \\) denotes taking the j’th coordinate of a tuple.

#### Poseidon Array hashing
Let \\( \text{hades}:\mathbb{F}_P^3\rightarrow\mathbb{F}_P^3 \\) denote the Hades permutation, with Starknet’s parameters, then given an array `a_1,...,a_n` of ***_𝑛_*** field elements we define `poseidon(a_1,...,a_n)` to be the first coordinate of `H(a_1,...,a_n;0,0,0)`, where: 
\\[
H(a_1,...,a_n;s_1,s_2,s_3)=\begin{cases}
H\big(a_3,...,a_n;\text{hades}(s_1+a_1, s_2+a_2, s_3)\big), & \text{if  } n\ge 2 \\
\text{hades}(s_1+a_1,s_2+1,s_3), & \text{if  } n=1 \\
\text{hades}(s_1+1,s_2,s_3), & \text{if  } n=0 \\
\end{cases}
\\]

#### Implementation

You can find an implementation of `Poseidon Hash` in Cairo 1 [here](https://github.com/starkware-libs/cairo/blob/1c02dc554e6923fbb1e24aaf3755bad115144712/corelib/src/poseidon.cairo#L65).

You can also try it in this code block, it is editable and runnable:
```rust,editable
use core::poseidon::PoseidonTrait;
use core::hash::{HashStateTrait, HashStateExTrait};

#[derive(Drop, Hash)]
struct StructForHash {
    first: felt252,
    second: felt252,
    third: (u32, u32),
    last: bool,
}

fn main() -> felt252 {
    let struct_to_hash = StructForHash { first: 0, second: 1, third: (1, 2), last: false };

    let hash = PoseidonTrait::new().update_with(struct_to_hash).finalize();
    hash
}

```

### L2 -> L1 Messaging
One of the key characteristics of a Layer 2 solution is its capacity to communicate with Layer 1.

Starknet employs a unique L1-L2 Messaging system, distinct from its consensus protocol and the process of submitting state updates on L1. This messaging system enables smart contracts on L1 to interface with those on L2 (and vice versa), facilitating "cross-chain" transactions.

![mechanism](https://images2.imgbox.com/da/8e/3MAMB2eC_o.jpeg)

In the `SNRC-20` standard, we use the Messaging System to send the content of the inscriptions to Ethereum L1. This allows the inscriptions to be verified on Ethereum as well, providing new possibilities for cross-chain operations of inscriptions.

However, when sending a message from Starknet to Ethereum, only the hash of the message is sent on L1 by the Starknet sequencer. You must write your own Ethereum contract to consume the message manually.

You can learn how to do it [here](https://book.cairo-lang.org/ch99-04-00-L1-L2-messaging.html).



## What is the next?

In the following chapters, we will explain in detail the specific format and calculation methods for each inscription operation. 
