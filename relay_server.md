##BCH Reusable Address Proposal: Ephemeral Relay Server##

This is a stub for the specs of communication protocol needed for the proposed Ephemeral Relay Server.

The initial subscription shall follow the [CashID](https://gitlab.com/cashid/protocol-specification) protocol, with the scan_pubkey as the desired identity, and the client specifying how long the subscription shall be - a standalone wallet client only looking to send transactions will have a shorter, ephemeral subscription. The Challenge Request can then be signed with the scan_privkey in a response. 

When implementing federation, a relay server can act as the intermediate and relay challenge messages between its clients and a remote relay server to start the subscription.

After a successful subscription, the server shall periodically challenge the server for responses to refresh the subscription. 

At sending, a logged in client will encrypt the transcription with the recipient's scan_pubkey and sent to the relay server. The relay server will hand it to the appropriate subscriber(s) as well as other relay servers; it returns an ack to the sending client upon relay, but the sending wallet shall not trust that message. The message is immediately destroyed upon successful relay. 
