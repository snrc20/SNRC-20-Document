# Transfer function Interface

The Transfer Interface defines a function interface that can execute a [Transfer operation](Transfer_OP.md).

## Input

There are no strict requirements for the input, but it is recommended to use the input format defined in the [Transfer operation](Transfer_OP.md).

You can also write a function that only takes the `tick`, `amt` and `recipient` parameters, then calculate the hash of all operations in your contract and inscribe it into the message, although this will consume more gas.

## Output
The output of this function should be a message payload, which conform to the following format:

|  Paramter   | type  | Description |  
|  ----  | ----  | ----  |
| Transfer_hash | Felt252 | Hash of Transfer operation |
| Sender | ContractAddress | Sender's address |
| Recipient | ContractAddress | Recipient's address |
| Amount | Felt252(u128) | Amount of this transfer operation |

These data contain the key elements to restore the complete `Transfer` inscription.

Here is an example of an message payload(in Hex):

|  Index   | Value  |
|  ----  | ----  | 
|  0  |  0x67c8925ab87c1501277c28168c59fbb93e434fd75c9133e455b866fd7efea5d | 
|  1  |  0x111111111111111111111111111111111111111111111111111111111111111 | 
|  2  |  0x0 | 
|  3	 | 0x22658 | 

## Event
After a Transfer function is successfully executed, it should emit an event. This will facilitate the indexer to include this transaction.

The format of the events is as follows:

`event Trasfer(sender, hash, recipient, amount)`

The parameters are:

|  Input   | Type  | Description |  
|  ----  | ----  | ----  |
|  sender | ContractAddress	  | Sender's address |
|  hash	| Felt252 | Mint_hash |
|  recipient | ContractAddress	  | Recipient's address |
|  amount	 | Felt252(u128)	 | Amount of this mint operation |