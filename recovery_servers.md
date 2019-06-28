##BCH Reusable Address Proposal: Recovery Server##

This is a stub for the specs of communication protocol needed for the proposed Recovery Server.

Client request: 

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | suffix_size | uint8 | length of the filtering suffix desired in bits; 0 if no-filter |
| variable | suffix | char | exact suffix used to filter transactions; pad with zero to whole bytes if suffix_size is not in steps of 8 |
| 4 | start height | uint32 | beginning block height, inclusive, where the client desires to be served transactions |
| 4 | pagesize | uint32 | maximum size, in bytes, the client desired to be served for the response. Excess transactions will need another request. | 

Server response:

The precise response and how/whether to paginate it is yet to be determined, but the server is expected to return a list of txids that contain the requested transactions. The client can download and verify these transactions from another service like Bitcore or ElectrumX. 

