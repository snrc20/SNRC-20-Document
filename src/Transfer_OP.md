# Transfer SNRC-20

## Off-Chain Calculation
Firstly, for any transfer operation, we need to prepare the following standard inscription format similar to [BRC-20](https://domo-2.gitbook.io/brc-20-experiment/), but without `amt`:
```json
{ 
  "p": "snrc-20",
  "op": "transfer",
  "tick": "tick"
}
```

At the same time, to be compatible with the [ETHS](https://docs.ethscriptions.com/overview/protocol-specification#how-to-validate-a-datauri) standard, we also need to satisfy the validation of `contentURI`. Therefore, the final Transfer inscription format involved in the calculation is as follows:
```json
data:,{ 
  "p": "snrc-20",
  "op": "transfer",
  "tick": "tick"
}
```

Next, we need to calculate the `Poseidon Hash` of the fixed part of the Transfer inscription. The complete Cairo program code is as follows(The following code block is editable and runnable) :

```rust,editable
use core::serde::Serde;
use core::poseidon::PoseidonTrait;
use core::byte_array::ByteArrayTrait;
use core::hash::{HashStateTrait, HashStateExTrait};
use core::array::{ArrayTrait, SpanTrait};
use core::traits::{Into, TryInto};

fn main() -> felt252 {
    /// Replace this with your own data. Be careful you should stringify it first.
    let tick = 'nwhp';
    let payload_pref: ByteArray = "data:,{\"p\":\"snrc-20\",\"op\":\"transfer\",\"tick\":\"";
    let payload_remain: ByteArray = "\"}";
    let mut output_array = ArrayTrait::<felt252>::new();
    let mut i: usize = 0;
    /// This is the same as `let serialized = payload.serialize(ref output_array);`
    /// , but it still not supported by the online compiler
    ///
    loop {
        if i >= payload_pref.len() {
            break;
        }
        let char = payload_pref.at(i).unwrap();
        char.serialize(ref output_array);
        i += 1;
    };
    tick.serialize(ref output_array);
    let mut i: usize = 0;
    loop {
        if i >= payload_remain.len() {
            break;
        }
        let char = payload_remain.at(i).unwrap();
        char.serialize(ref output_array);
        i += 1;
    };
    core::poseidon::poseidon_hash_span(output_array.span())
}

```

The return value of this Cairo program is 

`3186081088044837381540966151676090343793197013422856004776743671156846440042`, 

represented in hex as 

`0x70B420BAF038B3D467D80FD313B0D2AEDBEB46B7157CED682649D4507292E6A`.

This is the hash value corresponding to the Mint operation. 
Through this hash value, we will be able to identify what operations the user has performed on chain in the indexer, and restore the complete inscription by searching the hash table.

## Transfer an inscription by a SNRC-20 Contract

When users transfer in a contract that meets the `SNRC-20` standard, they need to organize their input in the following format:

|  Paramter   | type  | Description |  
|  ----  | ----  | ----  |
| Transfer_hash  | Felt252 | Hash of Transfer operation |
| Sender  | Felt252(address) | Address of the sender |
| Recipient  | Felt252(address) | Address of the recipient |
| Amount | Felt252(u128) | Amount of transfer  |

For the Transfer operation, there is no `Lim` limit -- of course, you cannot transfer more than the amount of inscription balance you own.

Therefore, the input value in this example coulb be:
```json
"Transfer_hash": "0x70B420BAF038B3D467D80FD313B0D2AEDBEB46B7157CED682649D4507292E6A"
"Sender": "Sender's address"
"Recipient": "Recipient's address"
"Amount": 19770525
```

Then, the contract that complies with the `SNRC-20` standard will inscribe the above input into the `L2->L1` Message and emit an event.