##BCH Reusable Address Proposal##

BIP-???

v0.1, with supplementary server structure outlines

@im_uname, with material from Mark Lundeberg, discussion with Chris Pacia, Shammah Chancellor and Jonathan Silverblood.

**Problem statement**

Most of the Bitcoin Cash ecosystem today runs on payments to straight addresses that are hashes of public keys, whether in simple P2PKH or scripted P2SH. Addresses are pseudonymous, and can provide a good - though imperfect - level of privacy if the receiver uses a fresh address to transact every time. This, however, presents a major problem in that users have to choose major compromises between usability, privacy, security, recoverability and trustlessness.

This draft reusable address format, if widely adopted, seeks to provide a major improvement over existing systems in terms of net gain in all five areas, as well as more flexibility in choosing desirable compromises depending on usecases under one common format.

**Existing payment systems both in use and theoretical**

***Simple HD wallets***

Probably the most widely used format in Bitcoin Cash for both person-to-person and merchant transactions, HD wallets simply supply a new address to the sender for each transaction either through direct interactions (such as chat channels or face-to-face talks) or payment processing software. It provides a high amount of privacy as long as the communication channel is not public, as well as reasonable security and recoverability, as the HD seed is easy to store and make redundant.

However, this method delivers a considerable compromise in usability: For it to operate as intended, the parties must interact for a given transaction, which is more difficult for usecases such as donations and casual transactions without fumbling for phones. More importantly, this is difficult to execute if one attempts to translate addresses into easy-to-recognize payment handles, where the sender has to find either an intermediary or the recipient's wallet and fetch a new address for every transaction.

***Single-address***

Widely used in donation usecases, single-address recipients provide high transparency and easy usability: The recipient does not have to be online to receive funds. It is also very easily represented by consumer-friendly handles due to one-to-one mappability, as currently implemented by [CashAccounts](https://gitlab.com/cash-accounts/specification).

While it provides good usability, security and recoverability, single-address use is terrible in privacy, as one published address allows every person who has ever transacted with the recipient to know all of her other transactions. As privacy often also implicates real life safety, this is highly undesirable in the quest to improve usability.

***Trusted relaying intermediaries***

In light of the complications associated with the currently used systems above, some solutions such as OpenCAP and HandCash seek to provide intermediate, always-online services that simply relay HD addresses to senders. This solves quite a few problems: Usability can be improved because the server is always online, hence easier to find via lookups from static handles; privacy is theoretically as good as HD wallets as long as one trusts the servers; and recoverability is the same as any seed-based solution.

However, introducing a third party to relay degrades both security and trustlessness. The third party can relay false addresses to senders, resulting in theft, and a service that serves a large number of clients can also be hacked, resulting in massive loss of service and money. Furthermore, privacy is always trusted to the server, and existing solutions generally rely on DNS as an identifier to senders, making it difficult for recipients who wish to remain trustlessly private to run their own servers. As time goes on, we can expect such services to concentrate in fewer and fewer hands, where their trusted nature will increase the attack surfaces for both security and privacy even if the operators remain well-intented.

***BIP-47 Payment codes***

[BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki), proposed by Justus Ranvier, is a radical divergence from the traditional recipient-hands-address-to-sender model. Rather, recipients publish a single public key that any sender can read. The sender then first publishes a "notification" that tells the recipient of the sender's public key, either through sending to a public notification address or through offchain channels, followed by an actual funding transaction whose receiving address is derived via Diffie-Hellman to be only identifiable by the sender and recipient. If used in an ideal setup, it theoretically offers great improvements at all fronts: Usability is improved due to the use of single identifiers, privacy is great among repeat transacting parties, security is not compromised, and recoverability from seed is possible in the case of public notification.

When weighed against real life usecases and wallets, BIP47 does, however, present several challenges:

1. One has to choose either degraded privacy or degraded recoverability. If notification is public, its privacy is degraded for a majority of transactions. Most bitcoin transactions are between parties who interact for the first time; recurring transactions are rare, and may remain so for a long time until worldwide adoption - a public notification allows for trivial timing correlation of transactions, narrowing down possible transactions for any given recipient to a trivially small subset. If one opts for offchain notification, and if the recipient loses backup of the public key notifications - typically harder to store and remain secure than seed words - then funds for the wallet may not be recoverable.

2. Implementation is complicated: If notification is onchain, a given wallet has to send two transactions to a first time recipient, and possibly obfuscate timing if one wants to have any semblance of privacy. This greatly complicates implementation difficulty, and creates additional problems in optimizing user experience, not to mention creating a disincentive to use such a system due to additional transaction fees. Offchain notifications lessen the burden, but then raise other questions in terms of recoverability (see above) as well as spam control.

***BIP-Stealth***

Originally proposed by [Peter Todd](https://github.com/genjix/bips/blob/master/bip-stealth.mediawiki) and later implemented by Chris Pacia, BIP-Stealth seeks a different path from BIP-47 in that it directly encrypts and embeds the sender's public key "notification" for each transaction in an op_return of the funding transaction, as well as attaching a prefix that allows the recipient to filter for his own transactions. The specification excels in its relative simplicity of implementation - there is only one transaction - as well as remaining recoverable from seed due to the fact that notifications are embedded, not to mention remaining trustless and secure. In fact, this proposal will reuse many of the same techniques as BIP-Stealth.

However, BIP-Stealth is still not ideal for several reasons:

1. Small anonymity set: Unless transactions across the ecosystem widely adopt BIP-stealth, transactions with an attached Stealth OP_RETURN will remain a very small subset of all transactions. In practice, this will mean a low level of privacy for its users for the foreseeable future, as users will have their transactions trivially identified from both timing and prefixes.

2. Incompatible with other OP_RETURN protocols: While this may be addressed in a future change in standardness rules, right now Bitcoin Cash only allows one OP_RETURN output per transaction. Embedding notifications inside OP_RETURN prevents the transaction from carrying other protocols such as Simple Ledger or CashIntents, limiting extensibility of the stealth transactions.

3. Flexibility in scaling: While BIP_Stealth does provide some flexibility in anonymity sets via adjusting prefix lengths, it does not provide means for low-bandwidth/trusted-privacy alternatives in offchain notification, nor does it provide for an expiry notice for clients who might want to update the address for scalability or privacy reasons periodically.

**Highlights of features in this proposal**

Usability and implementation ease: One single specification, two ways to transact - with the offchain method sharing a common gateway format for senders to minimize wallet burden while maximizing flexibility on receiving side.

Funds sent in one transaction - no setup. Possible second clawback transaction if not acknowledged in case of offchain method.

Privacy: Transaction indistinguishable from "normal" p2pkh/p2sh-multisig transactions, and has anonymity sets approximating a fraction of all transactions defined by the specified suffix.

Recoverability: Funds in the entire wallet recoverable by only seed phrases, with the help of a recovery server that does not have to compromise privacy.

Usability: Can receive from P2PKH or P2SH-multisig addresses, covering the vast majority of usecases. Theoretically possible to receive from any P2SH script with a pubkey and signature, but might result in wallet implementation complications.

Usability: Can receive to P2PKH or P2SH-multisig addresses, with payment codes adjusted accordingly.

Security: None of the servers, even "trusted" retention servers, have the ability to redirect or steal funds. The worst they can do is denial of service.

Optional retirement: Ability for addresses to "expire" and adjust resource usage after a long period. Also useful for addresses where the recipient intends to stop monitoring after a period of time for other reasons.

**Paycode format**

For a recipient who intends to receive to a p2pkh address, encode the following in bech32 using the same character set as cashaddr:

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | version | uint8 | paycode version byte; 1 for p2pkh. Set the first bit to 1 to indicate offline-communication only. |
| 1 | suffix_size | uint8 | length of the filtering suffix desired in bytes; 0 if no-filter for full-node or offline-communications |
| 32 | scan_pubkey | char | ECDSA/Schnorr public key of the recipient used to derive common secret |
| 32 | spend_pubkey | char | ECDSA/Schnorr public key of the recipient used to derive common secret |
| 4 | expiry | uint32 | UNIX time beyond which the paycode should not be used. 0 for never |
| 4 | checksum | char | last four bytes of SHA256 of all the preceding bytes. Intended for error-checking. |

For a recipient who intends to receive to a p2sh-multisig address, encode the following in bech32 using the same character set as cashaddr:

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | version | uint8 | paycode version byte; 2 for p2sh-multisig. Set the first bit to 1 to indicate offline-communication only. |
| 1 | suffix_size | uint8 | length of the filtering suffix desired in bytes; 0 if no-filter for full-node or offline-communications |
| 1 | multisig_setup | uint8 | instruction on constructing the multisig m-of-n to be paid to. first four bits indicate m parties who can recover funds, while last four bits indicate n parties total. Neither can be zero, m <= n. |
| 32 | scan_pubkey | char | ECDSA/Schnorr public key of the recipient used to derive common secret |
| 32 | spend_pubkey1 | char | First ECDSA/Schnorr public key of the recipients |
| 32 | spend_pubkey2 | char | Second ECDSA/Schnorr public key of the recipients |
| ... | ... | ... | ... |
| 32 | spend_pubkeyn | char | nth ECDSA/Schnorr public key of the recipients |
| 4 | expiry | uint32 | UNIX time beyond which the paycode should not be used. 0 for never |
| 4 | checksum | char | last four bytes of SHA256 of all the preceding bytes. Intended for error-checking. |

The payment code shall be prefixed with `paycode:`, and can be suffixed with offchain communications networks it supports in URI, e.g. `?xmpp=johndoe@something.org&matrix=@john123:something.com`. If no additional suffix is detected, the default offchain relay method is Ephemeral Relay service (see below).

**Paycode creation from two keypairs (P2PKH)**

Obtain two ECDSA/Schnorr keypairs from a wallet, and designate one as the "scanning" pubkey and the other as the "spending" pubkey.

Any common-secret-derived keypairs detected are additionally stored in the wallet file for history and spending, to minimize future bandwidth and CPU usage.

**Paycode creation (P2SH-multisig)**

The multisig m-of-n parties do not keep transaction scanning privacy from each other, and must agree on a common scanning keypair. They can subsequently submit one ECDSA/Schnorr spending pubkey each, and set up the paycode using the n+1 public keys.

**Generating a transaction to payment code (P2PKH)**

Sender's wallet shall first check the expiry time embedded in the paycode is at least one week ahead of local clock (skip if expiry time is 0). If expiry time is more than a week ahead, proceed.

Sender's wallet shall determine the outpoints she wants to spend from, and determine which input is at position 0.

In the case of paying from P2PKH, P below is simply the public key of the first input. In the case of paying from P2SH-multisig, it is the first public key with a valid signature.

A common secret c can be derived as follows:

Q = scan_pubkey

d = scan_privkey

R = spend_pubkey

f = spend_privkey

Q = dG

R = fG

P = eG = first public key embedded in input 0 with a valid signature

s = integer derived from outpoint spent by first input

Common secret c = H(H(eQ) + s) = H(H(dP) + s)

Pay to address from key R' = R + cG

To recap, we use the first pubkey with a valid signature of the transaction's first input, together with the scan key, to derive a shared secret via Diffie-Hellman. This shared secret is combined with the outpoint to obtain a unique scalar value for this payment that is used to tweak the spend key into a unique ephemeral key.

And then, use different nonces to sign the first input until the last suffix_size bytes of the signature are shared with the scan_pubkey (skip if suffix_size = 0). The payment transaction is then constructed and ready to be relayed.

**Generating a transaction to payment code (P2SH-Multisig)**

In the case of paying from P2PKH, P below is also the public key of the first input. In the case of paying from P2SH-multisig, it is the first public key with a valid signature.

The different part is all of the spending public keys will have to be involved in constructing a new P2SH address, described below in paying to a two-of-three P2SH-multisig setup:

Common secret c can be derived as follows:

Q = scan_pubkey

d = scan_privkey

R1, R2, R3 = spend_pubkey

f1, f2, f3 = spend_privkey

Q = dG

R1 = f1G, ...

P = eG = first public key embedded in input 0 with a valid signature

s = integer derived from outpoint spent by first input

Common secret c = H(H(eQ) + s) = H(H(dP) + s)

Pay to a new P2SH address constructed from keys R1' = R1 + cG, R2' = R2 + cG and R3' = R3 + cG, with m of n specified in OP_CHECKMULTISIG script.

Like the case of sending to P2PKH, the sender uses different nonces to sign the first input until the last suffix_size bytes of the signature are shared with the scan_pubkey (skip if suffix_size = 0). The payment transaction is then constructed and ready to be relayed.

**Relaying: Infrastructure needed**

There are two methods of receiving: ***Offchain communications***, which saves on bandwidth but entrusts privacy to relay and retention servers, and ***onchain direct sending***, which is trustless on privacy but requires more bandwidth. We will describe three types of servers needed:

1. ***Recovery servers***: In case of onchain direct sending, only this type of server is needed. The server shall index all transactions that have P2PKH or P2SH-multisig inputs by the last bytes of their first signature at the first input, and return all transactions matching requested signature suffixes to a client. Required for security anyway in the case of offchain communications. See recovery_server.md for specifications.

2. ***Ephemeral Relay servers***: This type of server can be freely signed up for using only public keys as identity, authenticating using signed messages as seen in CashID, and can employ basic rate controls against clients as seen in CashShuffle servers and remain largely permissionless. Simplistically relays encrypted messages from one pubkey identity to another. We can start with one central relay server, and gradually expand to multiple federated servers that share communications - a discovery mechanism should be in place similar to other decentralized applications. Does not store information for any significant lengths of time, stateless, and mostly consumes only bandwidth. See relay_server.md for specifications.

3. ***Retention servers***: This type of server retains transaction information for offline clients, and can be permissioned while allowing significant innovation and profit models at scale. Entrusted with client scan keys, retention servers connect to relay servers for their clients, then retrieve, decrypt and broadcast transactions for them. They are also responsible for retaining txid information for easy retrieval when client reconnects. Operators of Retention Servers can be expected to also operate their own Ephemeral Relays federated with other operators. An example that takes advantage of CashAccounts can be found at cashaccounts_retention.md.

**Sending: Onchain direct sending**

After a transaction is generated, if the sending wallet detects the first bit of the version byte is zero, it can simply broadcast the transaction to the Bitcoin Cash network and let it be mined. No notification to the recipient is needed.

**Sending: Offchain communications**

If the paycode specifies offchain communications via setting the first bit to 1 and does not specify additional relay methods, the sending wallet shall attempt to relay through Ephemeral Relay servers. The constructed transaction shall first be encrypted with the payment code's scan pubkey, then handed off to a relay server. The transaction is considered "Sent" when the sending wallet detects the same transaction being broadcasted by the retention server.

To remain trustless against the possibility of relay or retention servers denying service, after a short timeout (e.g. 30 seconds), if the transaction is not detected, the sending wallet shall consider the broadcast failed and construct a "clawback" transaction that spends the same output to a new address she controls. This is to avoid the case where the trade is voided - recipient never sends goods or services to the sender - yet after some time the recipient broadcasts the transaction anyway, robbing the sender.

**Receiving: Onchain direct**

If onchain direct sending is used, receiving is relatively straightforward. The recipient shall connect to a Recovery Server and attempt to download all transactions that match his payment code's suffix since he was last online. This will cost bandwidth that is at most 1/256 of downloading the full blockchain (in the case suffix length = 1 byte), and less if longer suffix length is specified.

Upon receiving subscribed transactions, the wallet can then attempt, for each transaction, to derive common secret c from scan_privkey, outpoint spent by the first input and the first public key from the first input of each suffixed transaction. Discard transactions where addresses do not match R' = R + cG, in a similar fashion as two-key BIP-Stealth. This step can also be performed by specialized, trusted recovery servers entrusted with scan keys.

Upon obtaining transactions filtered by the scan_privkey, the receiving wallet then stores it locally and assign a spending keypair R' and h = (f+c) to it. Funds are now available to be spent.

**Receiving: Offchain communications**

A Retention Server, subscribing to Relays using a client's scan privkey, receives the encrypted transaction, then decrypts and broadcasts to the Bitcoin Cash network; invalid transactions can be discarded. The server then proceeds to store the relevant txids calculated from the broadcasted transaction for its clients until retrieved - retention time can vary depending on provider, from several weeks to indefinite depending on specific quality of service desired.

Clients can log onto retention servers and retrieve their incoming txids. Depending on the level of service, the login method can be relatively permissionless (depending on CashAccounts, for example; see cashaccounts_retention.md) or permissioned in premium or other methods. Specific arrangements depend on the exact setup and can even be extended to use other communications protocols such as XMPP and Tox, to be determined by the implementing wallet.

After a client fetches a txid from the retention server, he should proceed to request the full transaction from a node, and extract spending keypair as described above. Then the transaction and spending keypair for its output should be stored locally.

Depending on the specific setup, if a client suspects either server downtime, malicious denial of service, or expired retention, it shall connect to a Recovery Server and attempt to recover funds as described in Onchain Direct transactions.

**Receiving to P2SH-Multisig**

The scanning and filtering part shall work exactly like in P2PKH. Once the multisig parties have a filtered transaction ascertained by the scan_privkey, the receiving parties can then each assign public keys R1', R2'... and private keys f1+c, f2+c...to them. With these keysets, the address can be spent from normally.

**Expiration time**

Whether it uses onchain direct sending or offchain communications, as long as the wallet remains compatible with seed recovery via recovery servers, it will require a fixed fraction of total bitcoin cash bandwidth for that purpose. While the consumption does not have to be latency sensitive in the context of a light wallet, it is vulnerable to long term total traffic fluctuation. If the suffix is too short while network traffic gets much higher, the wallet might have difficulty running as bandwidth requirement rises with network traffic. On the other hand, if the suffix is too long when network traffic is low, the wallet's privacy is degraded as its anonymity set shrinks.

In order to mitigate this, an optional expiry time can be added; sending wallets shall respect the expiry time by yielding an error if attempting to send past it - any funds that are sent overriding the expiry is considered lost, and the recipient has done due diligence warning the sender in the paycode.

When scanning, nodes or wallets can allow for a certain amount of buffer beyond the expiry date, in case a transaction is sent before but remains unconfirmed beyond the expiry - in the context of BCH, a week might be more than sufficient.

In addition to addressing scaling concerns, expiration also addresses another usecase complaint - that wallets once established will have to monitor addresses indefinitely, and that there is no clear indicator to a sender whether an address remains usable or not. Such a clear guide embedded in the address itself can serve these cases well and provide unambiguous dates beyond which recipient will be free from the burden of keeping monitor and keys.

**Considerations**

Anonymity set for suffix_size 0 is effectively all transactions with p2pkh outputs (TBD p2sh-multisig); with suffix_size > 0, it is reduced by a factor of 1/(256)^(suffix_size) at each step.
