##BCH Reusable Address Proposal: Recovery Server##

This is a stub for the specs of communication protocol needed for the proposed Recovery Server.

Method ideas:

**version** 

Simple version handshake. Client sends a version request with his own desired version, server returns with two values: min_version and max_version, which indicates features it supports. 

**get_history**

Client requests: 

1. Version
2. Start_block 
3. End_block (0 if including unconfirmed)
4. Prefix length
5. Desired prefix
6. Flag

Server rejects if version unsupported. 

Otherwise server first returns an estimated size of transactions (or txids, if compact) to transmit. Client acknowledges, then between start_block and end_block inclusive, return a list of transactions with at least one qualifying input's double sha256 hash matching the desired prefix string and length, each accompanied by their block number if confirmed. Returns error if rate-limited.

Flag can be used to request utxo (returns only unspent) and/or compact (returns txid instead of whole transactions).

**subscribe**

Client request: 

1. Version
2. Prefix length
3. Desired prefix
4. Expiry time
5. Flag

Server monitors mempool + newly confirmed blocks, and pass any matching transactions (see get_history) to client. Server stops monitoring at expiry time, can be renewed. Rejects if expiry time is too far in the future. 

Flag can be used to request compact (returns txid instead of whole transactions). 

-- optional --

**get_peers** 

get a list of known server peers (host/ports) along with their min_version and max_versions. Useful for federation. 

**add_peer** 

A server announces its existence to other servers along with its min/max versions, which indicates features it support. Useful for federation. Peers may periodically check each other for integrity using minimal get_history. 

**paid_history** 

Like get_history, but server returns a paycode and amount along with the size estimate. Instead of a simple acknowledgement, client returns a transaction, which the server parses and broadcasts before serving data. Returns error if payment invalid or insufficient.

**paid_subscribe**

Like subscribe, but with an additional handshake of paycode and amount. Client returns a transaction as in paid_history before the monitoring service is activated

