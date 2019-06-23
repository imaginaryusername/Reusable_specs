##BCH Reusable Address Proposal: Recovery Server##

This is a stub for the specs of communication protocol needed for the proposed Recovery Server.

Client request: 

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | suffix_size | uint8 | length of the filtering suffix desired in bytes; 0 if no-filter |
| variable | suffix | char | exact suffix used to filter transactions |
| 4 | start time | uint32 | Beginning UNIX time of the period of transactions the client wishes to examine |
| 4 | end time | uint32 | End UNIX time of the period of transactions the client wishes to examine | 

Server response:



