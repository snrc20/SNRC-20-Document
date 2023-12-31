# Deploy function Interface

The Deploy Interface defines a function interface that can execute a [Deploy operation](Deploy_OP.md).

## Input

There are no strict requirements for the input, but it is recommended to use the input format defined in the [Deploy operation](Deploy_OP.md).

You can also write a function that only takes the `tick`, `max`, `lim` parameters, then calculate the hash of all operations in your contract and inscribe it into the message, although this will consume more gas.

## Output
The output of this function should be a message payload, which conform to the following format:

|  Paramter   | type  | Description |  
|  ----  | ----  | ----  |
| Deploy_hash  | Felt252 | Hash of Deploy operation |
| Mint_hash | Felt252 | Hash of Mint operation |
| Transfer_hash | Felt252 | Hash of Transfer operation|
| Tick | Felt252(ShortString) | Ticker: A string identifier of the snrc-20 |
| Max | Felt252(u128) | Max supply: Max supply of the snrc-20 |
| Lim | Felt252(u128) | Mint limit: Limit per tx for users |

These data contain the key elements to restore the complete `Deploy` inscription, and provide the Hash for `Mint` and `Transfer`, which will facilitate indexing.

Here is an example of an message payload(in Hex):

|  Index   | Value  |
|  ----  | ----  | 
|  0  |  0x206E97BD728106AAE642A8847107EF91922321FF00A4FEB7A99880D4D8BA962 | 
|  1	 | 0x33FF744581AA76AFA81006908ADC9A41B68FACB15ECF5E980EF56F9910380DE | 
|  2	 | 0x70B420BAF038B3D467D80FD313B0D2AEDBEB46B7157CED682649D4507292E6A | 
|  3 | 	0x434f4f4c | 
|  4 | 	0x22658 | 
|  5 | 	0x22658 | 

## Event
After a Deploy function is successfully executed, it should emit an event. This will facilitate the indexer to include this transaction.

The format of the events is as follows:

`event Deploy(sender, hash, tick, max, lim)`

The parameters are:

|  Input   | Type  | Description |  
|  ----  | ----  | ----  |
|  sender | ContractAddress	  | Sender's address |
|  hash	| Array::< Felt252 > | [Deploy_hash, Mint_hash, Transfer_hash] |
 |tick	 | Felt252(ShortString) |  A string identifier of the snrc-20  |
 |max	 | Felt252(u128)	 | Max supply: Max supply of the snrc-20 |
 |lim	 | Felt252(u128)	 | Mint limit: Limit per tx for users |
