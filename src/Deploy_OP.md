# Deploy SNRC-20

## Off-Chain Calculation

Consider the following inscription format, similar to [BRC-20](https://domo-2.gitbook.io/brc-20-experiment/):
```json
{ 
  "p": "snrc-20",
  "op": "deploy",
  "tick": "COOL",
  "max": "140888",
  "lim": "140888" 
}
```

It's important to note that, since the default type of the Cairo virtual machine is `Felt252`, all numerical parameters should be of the u128 type, which ranges from `0` to `2^128-1` and can fit within 252 bits. The type requirements are as follows:

|  Key   | type  | Description |  
|  ----  | ----  | ----  |
| p  | Felt252(ShortString) | Protocol: Helps other systems identify and process snrc-20 events |
| op | Felt252(ShortString) | Operation: Type of event (Deploy, Mint, Transfer) |
| tick | Felt252(ShortString) | Ticker: A string identifier of the snrc-20 |
| max | Felt252(u128) | Max supply: set max supply of the snrc-20 |
| lim | Felt252(u128) | Mint limit: Limit per tx for users |

At the same time, to be compatible with the [ETHS](https://docs.ethscriptions.com/overview/protocol-specification#how-to-validate-a-datauri) standard, we also need to satisfy the validation of `contentURI`. 

Therefore, the final Deploy inscription format involved in the calculation is as follows:

```json
data:,{ 
  "p": "snrc-20",
  "op": "deploy",
  "tick": "COOL",
  "max": "140888",
  "lim": "140888" 
}
```

Next, we need to calculate the `Poseidon Hash` of the fixed part of the Mint inscription. The complete Cairo program code is as follows(The following code block is editable and runnable) :

```rust,editable
use core::clone::Clone;
use core::serde::Serde;
use core::poseidon::PoseidonTrait;
use core::byte_array::ByteArrayTrait;
use core::hash::{HashStateTrait, HashStateExTrait};
use core::array::{ArrayTrait, SpanTrait};
use core::traits::{Into, TryInto};

fn main() -> felt252 {
    /// Replace this with your own data. Be careful you should stringify it first.
    let tick = 'COOL';
    let max: u128 = 140888;
    let lim: u128 = 140888;
    let payload_pref: ByteArray = "data:,{\"p\":\"snrc-20\",\"op\":\"deploy\",\"tick\":\"";
    let payload_max: ByteArray = "\",\"max\":\"";
    let payload_lim: ByteArray = "\",\"lim\":\"";
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
    i=0;
    loop {
        if i >= payload_max.len() {
            break;
        }
        let char = payload_max.at(i).unwrap();
        char.serialize(ref output_array);
        i += 1;
    };
    i=0;
    max.serialize(ref output_array);
    loop {
        if i >= payload_lim.len() {
            break;
        }
        let char = payload_lim.at(i).unwrap();
        char.serialize(ref output_array);
        i += 1;
    };
    i=0;
    lim.serialize(ref output_array);
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

`3539966466338062280723903945602743530625753975496839308572805391711777814555`

, represented in hex as 

`0x7d38ccde6d4bc0efda3e101b134f7806e37b03894aa41afa9fe0268687f781b`.

This is the hash value corresponding to the Deploy operation. 

Through this hash value, we will be able to identify what operations the user has performed on chain in the indexer, and restore the complete inscription by searching the hash table.

## Organize parameters for Deploy operartion
However, for the complete Deploy parameters, we also need the hash values corresponding to the inscriptions of Mint and Transfer operations. 

Please refer to the next two chapters for this content. Here, for this tick : 
```
  "tick": "COOL",
  "max": "140888",
  "lim": "140888" 
```
We will first assume that the hash corresponding to Mint is 

`0x59b6615aedfdd1bd23b762b51485eae4c5ee84189c21c2b59822fea62518eb5`

, and the hash corresponding to Transfer is 

`0x67c8925ab87c1501277c28168c59fbb93e434fd75c9133e455b866fd7efea5d`.

A Deploy operation requires the following six parameters:

|  Paramter   | type  | Description |  
|  ----  | ----  | ----  |
| Deploy_hash  | Felt252 | Hash of Deploy operation |
| Mint_hash | Felt252 | Hash of Mint operation |
| Transfer_hash | Felt252 | Hash of Transfer operation|
| Tick | Felt252(ShortString) | Ticker: A string identifier of the snrc-20 |
| Max | Felt252(u128) | Max supply: Max supply of the snrc-20 |
| Lim | Felt252(u128) | Mint limit: Limit per tx for users |

Where `deploy_hash`, `mint_hash`, `transfer_hash` are the `Poseidon Hash` corresponding to the Deploy, Mint, and Transfer inscriptions respectively.


## Deploy an inscription by a SNRC-20 Contract

When users deploy an inscription in a contract that meets the `SNRC-20` standard, they need to organize their input in a format like previous section said.

The input value in this example should be:

```json
"Deploy_hash": "0x7d38ccde6d4bc0efda3e101b134f7806e37b03894aa41afa9fe0268687f781b"
"Mint_hash": "0x59b6615aedfdd1bd23b762b51485eae4c5ee84189c21c2b59822fea62518eb5"
"Transfer_hash": "0x67c8925ab87c1501277c28168c59fbb93e434fd75c9133e455b866fd7efea5d",
"Tick": "COOL",
"Max": 140888,
"Lim": 140888
```

Then, the contract that complies with the `SNRC-20` standard will inscribe the above input into the `L2->L1` Message and emit an event.

For more information about the SNRC-20 Contract, please refer to [this chapter](SNRC_20_Contract.md).

