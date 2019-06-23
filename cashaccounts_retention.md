##BCH Reusable Address Proposal: Retention Server (Cashaccount)##

This is a stub for the specs of an example Retention Server based on Cashaccounts.

The server shall retain transaction ids for its client in a relatively permissionless way; to subscribe for its service, a client will simply need to supply a Cashaccount registration, along with the scan privkey. To maximize DoS resistance efficiency, first implementations of this retention server can be coupled with the relay server as well, opening relays to other types of retention servers as well as personal subscribers. Future implementations can be more decoupled, with retention servers acquiring trusted tokens from relays to allow for higher subscription bandwidth. 
