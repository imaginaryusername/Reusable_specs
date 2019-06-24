##BCH Reusable Address Proposal: Recovery Server##

This is a stub for the specs of communication protocol needed for the proposed Recovery Server.

Client request: 

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | suffix_size | uint8 | length of the filtering suffix desired in bits; 0 if no-filter |
| variable | suffix | char | exact suffix used to filter transactions; pad with zero to whole bytes if suffix_size is not in steps of 8 |
| 4 | start time | uint32 | Beginning UNIX time of the period of transactions the client wishes to examine |
| 4 | end time | uint32 | End UNIX time of the period of transactions the client wishes to examine | 

Server response:

The precise response and how/whether to paginate it is yet to be determined, but the server is expected to return a list of txids that contain the requested transactions. The client can download and verify these transactions from another service like Bitcore or ElectrumX. 

