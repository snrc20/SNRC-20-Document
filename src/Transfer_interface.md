# Transfer function Interface

The Transfer Interface defines a function interface that can execute a [Transfer operation](Transfer_OP.md).

## Input

There are no strict requirements for the input, but it is recommended to use the input format defined in the [Transfer operation](Transfer_OP.md).

You can also write a function that only takes the `tick`, `amt` parameters, then calculate the hash of all operations in your contract and inscribe it into the message, although this will consume more gas.

## Output
The output of this function should be a message payload, which conform to the following format:

|  Paramter   | type  | Description |  
|  ----  | ----  | ----  |
| Transfer_hash | Felt252 | Hash of Transfer operation |
| Amount | Felt252(u128) | Amount of this Transfer operation |

These data contain the key elements to restore the complete `Transfer` inscription.

Here is an example of an message payload(in Hex):

|  Index   | Value  |
|  ----  | ----  | 
|  0  |  0x59b6615aedfdd1bd23b762b51485eae4c5ee84189c21c2b59822fea62518eb5 | 
|  1	 | 0x22658 | 

## Event
After a Transfer function is successfully executed, it should emit an event. This will facilitate the indexer to include this transaction.

The format of the events is as follows:

`event Transfer(sender, hash, amount)`

The parameters are:

|  Input   | Type  | Description |  
|  ----  | ----  | ----  |
|  sender | ContractAddress	  | Sender's address |
|  hash	| Felt252 | Transfer_hash |
| amount	 | Felt252(u128)	 | Amount of this Transfer operation |
