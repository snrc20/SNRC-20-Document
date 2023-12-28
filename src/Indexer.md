# Indexer

Since inscriptions do not adopt the account balance mechanism by default (reminder: you can still combine it with the account balance mechanism, this is not mandatory), you need an indexer to record user balances.

For a contract that complies with the `SNRC-20` standard, there are two ways to index: 

**Complete indexing** and **Quick indexing**.

## How to index?

### Complete Indexing
Complete indexing means that for an inscription transaction, you need to obtain its `Events`, `Transaction`, `TransactionReceipt` on the Starknet chain, and index them.

### Quick Indexing
Quick indexing allows you to only index `Events`, to establish an index that only contains basic inscription information. This is suitable for scenarios that do not need to read `L2->L1 Message`.

### Use Full-node or RPC service
You can obtain the necessary information through any Full-node or RPC service with an endpoint version greater than `0.5`. 
Here is a list of [available services](https://www.starknet.io/en/ecosystem/fullnodes-and-rpc-services).

You can refer to the [Voyager's](https://docs.data.voyager.online/refs) documentation to learn how to use Starknet's RPC service and find the APIs you need.

<?// TODO: Full example>

