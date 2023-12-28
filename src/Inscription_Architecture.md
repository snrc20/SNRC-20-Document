# SNRC-20 Design Philosophy

Due to the presence of many unnecessary parts in the traditional BRC-20 inscriptions, SNRC-20 adopts the approach of Off-Chain calculation and transmission of hashes for inscriptions. 


For example, a standard BRC-20 inscription requires the following three JSON objects to implement all its operations:

**Deploy**
```json
{ 
  "p": "brc-20",
  "op": "deploy",
  "tick": "ordi",
  "max": "21000000",
  "lim": "1000"
}
```

**Mint**
```json
{ 
  "p": "brc-20",
  "op": "mint",
  "tick": "ordi",
  "amt": "1000"
}
```

**Transfer**
```json
{ 
  "p": "brc-20",
  "op": "transfer",
  "tick": "ordi",
  "amt": "100"
}
```

We can see that there is a lot of repeated content in these JSON objects.

For example, if we want to deploy a BRC-20 based inscription that is different from the `ordi` inscription, we actually only need to change the following three fields:
```json
  "tick": new_tick,
  "max": max_supply,
  "lim": limit
```

The remaining parts are the same for all inscriptions of `brc-20` protocol, and we can easily restore the complete `Deploy` inscription json from the above three fields.

The same applies to `Mint` and `Transfer`.

Since we can infer the `Mint` or `Transfer` inscription json from the `Deploy` inscription json. We only need the `tick` and `amt` fields, along with the `Deploy` inscription json, to restore the complete `Mint` or `Transfer` inscription json.

Therefore, as mentioned above, only three fields are needed to restore the complete `Deploy` inscription. As long as we can query the complete `Deploy` inscription, we can restore the `Mint` and `Transfer` inscriptions from a few parameters. 

Now we can say that an inscription based on BRC-20 only requires the following JSON data:

```json
 {
  "tick": new_tick,
  "max": max_supply,
  "lim": limit
  }
```

**This is the initial design concept of SNRC-20.**

However, this is still not concise enough. Do we really need to transmit inscription's data in JSON format and inscribe it on the blockchain? 

As is well known, the Cairo language is not very friendly to `string` due to its type system. Can we design an inscription system that is more suitable for Starknet?

**We thought of hash.** Hash can be considered the lifeblood of the blockchain. Its uniqueness and verifiability provide great convenience for the blockchain. For unchanging content, using its hash is the best method for transmission, comparison, and indexing. Coincidentally, the inscription system has a lot of unchanging content.

Also, the default type `Felt252` of the Cairo language is designed to be suitable for storing hashes. Consider this approach:

>Convert the unchanging content into a hash, combine it with the changing content, and use as few `Felt252` as possible to transmit and inscribe data.

**This is the way. This is the design philosophy of SNRC-20.**

![This is the way](https://images2.imgbox.com/a7/a2/IeRo4kn9_o.png)