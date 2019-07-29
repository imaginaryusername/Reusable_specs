##Wallet implementation recommendations##

This is a stub for wallet behavior implementation recommendations. The spec is robust against deviations from the recommendations, but aligning user experience and expectations will be a plus across the ecosystem.

**Key derivation paths from BIP44 wallets**

For a wallet adhering to BIP44 standard of m/purpose'/coin'/account'/change/index, constant 0 is typically used for "external" chain and 1 for "internal" at the "change" position. One SHOULD additionally use 2 for scan_key and 3 for spend_key at "change" for the same wallet, and start at 0 for index; scan_key and spend_key of the same index numbers SHOULD be paired in generating reusable paycodes. 

To minimize monitoring load, only reusable paycodes that has been explicitly generated should be monitored for receives. 
