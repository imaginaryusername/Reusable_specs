# BCH Reusable Address Proposal

BIP-???

v0.4.3, further limit number of inputs

@im_uname, with material from Mark Lundeberg, plus discussion with Chris Pacia, Amaury Séchet, Shammah Chancellor, Jonathan Silverblood and Josh Ellithorpe. Additional editing from Freetrader, Emergent_reasons and Jonald Fyookball.

# Introduction

**Problem statement**

Most of the Bitcoin Cash ecosystem today runs on payments to straight addresses that are hashes of public keys, whether in simple P2PKH or scripted P2SH. Addresses are pseudonymous, and can provide a good - though imperfect - level of privacy if the receiver uses a fresh address to transact every time. Despite the existence or proposal of various alias/handle systems, there still exists a major problem in that users have to make compromises between usability, privacy, security, recoverability and trustlessness.

**Solution**

We propose a new alias system that would allow senders to generate a fresh address for any recipient with a handle. Communicating the existence of the transaction happens on-chain --actually embedded in the transaction itself, without using OP_RETURN.  This is accomplished by combining the Elliptic-curve Diffie-Helman properties of bitcoin keys with a simple grinding system, resulting in a byte-prefix that can be found by scanning while also hiding within an acceptable anonymity set.

This draft reusable address format, if widely adopted, seeks to provide a major improvement over existing systems in terms of net gain in all five areas, as well as more flexibility in choosing desirable compromises depending on usecases under one common format.

# Part I: Design Discussion


## Requirements

1. From only the paycode, sender can generate addresses that are detectable and spendable by the recipient.

2. Multiple payto addresses can be generated to a single recipient from a single notification, so amount can be obfuscated.

3. The sender can generate the recoverable payto address relevant to his payment, but cannot otherwise compromise the privacy or fund security of the recipient by deriving any of his private keys.

4. The transactions should have a reasonable anonymity set, where the recipient's transactions are not easily isolated on the blockchain.

5. The receiver must be able to separate the keys used to generate and detect addresses from the keys used to spend (which can be offline), so to separate the privacy and security aspects.

6. To remain light wallet compatible, the system must be able to reduce the number of transactions for the recipient to detect to a subset of all transactions, even at higher total transaction throughputs.

7. The transaction and notification must be self-contained, no additional transaction is needed to send a payment, no additional data other than the transaction itself is needed to receive and spend it. Derived addresses do not need to continue to be monitored after a transaction is detected.

8. There must exist a practical way for the recipient to recover his funds from mnemonic seed backups without compromising security or privacy.

9. Multisignature addresses must be supported for both sender and recipient.

10. Inputs and outputs can be in any order, so trustless coin mixing can be flexibly accommodated.

11. Compatible with other OP_RETURN protocols, which form an important part of the Bitcoin Cash ecosystem. Incompatibility may lead to low adoption or fragmented anonymity sets.

12. For offline notification methods, the intermediary servers must not be able to compromise security of funds.


## Existing payment systems both in use and theoretical

***Simple HD wallets***

Probably the most widely used format in Bitcoin Cash for both person-to-person and merchant transactions, HD wallets simply supply a new address to the sender for each transaction either through direct interactions (such as chat channels or face-to-face talks) or payment processing software. It provides a high amount of privacy as long as the communication channel is not public, as well as reasonable security and recoverability, as the HD seed is easy to store and make redundant.

However, this method delivers a considerable compromise in usability: For it to operate as intended, the parties must interact for a given transaction, which is more difficult for usecases such as donations and casual transactions without fumbling for phones. More importantly, this is difficult to execute if one attempts to translate addresses into easy-to-recognize payment handles, where the sender has to find either an intermediary or the recipient's wallet and fetch a new address for every transaction.

***Single-address***

Widely used in donation usecases, single-address recipients provide high transparency and easy usability: The recipient does not have to be online to receive funds. It is also very easily represented by consumer-friendly handles due to one-to-one mappability, as currently implemented by [CashAccounts](https://gitlab.com/cash-accounts/specification).

While it provides good usability, security and recoverability, single-address use is terrible in privacy, as one published address allows every person who is aware of her ownership of the address to know all of her other transactions. As privacy often also implicates real life safety, this is highly undesirable in the quest to improve usability.

***Trusted relaying intermediaries***

In light of the complications associated with the currently used systems above, some solutions such as OpenCAP and HandCash seek to provide intermediate, always-online services that simply relay HD addresses to senders. This solves quite a few problems: Usability can be improved because the server is always online, hence easier to find via lookups from static handles; privacy is theoretically as good as HD wallets as long as one trusts the servers; and recoverability is the same as any seed-based solution.

However, introducing a third party to relay degrades both security and trustlessness. The third party can relay false addresses to senders, resulting in theft, and a service that serves a large number of clients can also be hacked, resulting in massive loss of service and money. Furthermore, privacy is always trusted to the server, and existing solutions generally rely on DNS as an identifier to senders, making it difficult for recipients who wish to remain trustlessly private to run their own servers. As time goes on, we can expect such services to concentrate in fewer and fewer hands, where their trusted nature will increase the attack surfaces for both security and privacy even if the operators' intentions remain good.

***BIP-47 Payment codes***

[BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki), proposed by Justus Ranvier, is a radical divergence from the traditional recipient-hands-address-to-sender model. Rather, recipients publish a single public key that any sender can read. The sender then first publishes a "notification" that tells the recipient of the sender's public key, either through sending to a public notification address or through offchain channels, followed by an actual funding transaction whose receiving address is derived via Diffie-Hellman to be only identifiable by the sender and recipient. If used in an ideal setup, it theoretically offers great improvements at all fronts: Usability is improved due to the use of single identifiers, privacy is great among repeat transacting parties, security is not compromised, and recoverability from seed is possible in the case of public notification.

When weighed against real life usecases and wallets, BIP47 does, however, present several challenges:

1. One has to choose either degraded privacy or degraded recoverability. In the case of public notifications, privacy is degraded for a majority of transactions. Most bitcoin transactions are between parties who interact for the first time; recurring transactions are rare, and may remain so for a long time until worldwide adoption - a public notification allows for trivial timing correlation of transactions, narrowing down possible transactions for any given recipient to a trivially small subset. If one opts for offchain notification, and if the recipient loses their backup of any public key notifications - typically harder to store and keep secure than seed words - then funds for the wallet may not be recoverable.

2. Implementation is complicated: If notification is onchain, a given wallet has to send two transactions to a first time recipient, and possibly obfuscate timing if one wants to have any semblance of privacy. This greatly complicates implementation difficulty, and creates additional problems in optimizing user experience, not to mention creating a disincentive to use such a system due to additional transaction fees. Offchain notifications lessen the burden, but then raise other questions in terms of recoverability (see above) as well as spam control.

***BIP-Stealth***

Originally proposed by [Peter Todd](https://github.com/genjix/bips/blob/master/bip-stealth.mediawiki) and later implemented by Chris Pacia, BIP-Stealth seeks a different path from BIP-47 in that it directly encrypts and embeds the sender's public key "notification" for each transaction in an op_return of the funding transaction, as well as attaching a prefix that allows the recipient to filter for his own transactions. The specification excels in its relative simplicity of implementation - there is only one transaction - as well as remaining recoverable from seed due to the fact that notifications are embedded, not to mention remaining trustless and secure. In fact, this proposal will reuse many of the same techniques as BIP-Stealth.

However, BIP-Stealth is still not ideal for several reasons:

1. Small anonymity set: Unless transactions across the ecosystem widely adopt BIP-stealth, transactions with an attached Stealth OP_RETURN will remain a very small subset of all transactions. In practice, this will mean a low level of privacy for its users for the foreseeable future, as users will have their transactions trivially identified from both timing and prefixes.

2. Incompatible with other OP_RETURN protocols: While this may be addressed in a future change in standardness rules, right now Bitcoin Cash only allows one OP_RETURN output per transaction. Embedding notifications inside OP_RETURN prevents the transaction from carrying other protocols such as Simple Ledger or CashIntents, limiting extensibility of the stealth transactions.

3. Flexibility in scaling: While BIP-Stealth does provide some flexibility in anonymity sets via adjusting prefix lengths, it does not provide means for low-bandwidth/trusted-privacy alternatives in offchain notification, nor does it provide for an expiry notice for clients who might want to update the address for scalability or privacy reasons periodically.

## Highlights of features in this proposal

Usability: Sender does not require any additional information aside from the paycode. (REQ-1)

Usability and implementation ease: One single specification, two ways to transact - with the offchain method sharing a common gateway format for senders to minimize wallet burden, while maximizing flexibility on receiving side allowing them to entrust privacy to intermediaries. (REQ-5)

Funds sent in one transaction: No setup. Possible second clawback transaction if offchain notification is not acknowledged. (REQ-7)

Privacy: Transaction indistinguishable from "normal" p2pkh/p2sh-multisig transactions, and has anonymity sets approximating a fraction of all transactions defined by the specified prefix. Sender does not know other transactions sent to the recipient. (REQ-3,4)

Privacy: For transactions with multiple inputs and outputs, it will be unclear to an observer which input is intended to be used for filtering as well as which output(s) are intended for the recipient (REQ-2,10)

Recoverability: Funds in the entire wallet recoverable by only seed phrases, with the help of a recovery server that does not have to compromise privacy. (REQ-8,6)

Usability: Can receive from P2PKH or P2SH-multisig addresses, covering the vast majority of usecases. Theoretically possible to receive from any P2SH script with a pubkey and signature, but might result in wallet implementation complications. (REQ-9)

Usability: Can receive to P2PKH or P2SH-multisig addresses, with payment codes adjusted accordingly. (Req 9)

Usability: Compatible with OP_RETURN protocols such as Simple Ledger Protocol, CashIntents or OMNI without adjusting current network protocol. (REQ-11)

Security: None of the servers, even "trusted" retention servers, have the ability to redirect or steal funds. The worst they can do is denial of service, which can be circumvented by falling back to recovery servers. (REQ-12)

Optional retirement: Ability for addresses to "renew" by expiring and republishing with adjusted resource usage after some period. Also useful for addresses where the recipient intends to stop monitoring after a period of time for other reasons. (REQ-6 related)

# Part II: Proposal Details

## Paycode format

For a recipient who intends to receive to a p2pkh addresses, encode the following in base32 using the same character set as cashaddr:

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | version | uint8 | paycode version byte; 1 and 2 for p2pkh (mainnet), 5 and 6 for p2pkh (testnet), among them 2 and 6 to force offline-communication only. |
| 1 | prefix_size | uint8 | length of the filtering prefix desired, 0, 4, 8, 12 or 16 bits for versions 1 through 8 ; 0 if no-filter for full-node or offline-communications. If used, recommend >= 8. |
| 33 | scan_pubkey | char | 256-bit compressed ECDSA/Schnorr public key of the recipient used to derive common secret |
| 33 | spend_pubkey | char | 256-bit compressed ECDSA/Schnorr public key of the recipient used to derive payto addresses when combined with common secret |
| 4 | expiry | uint32 | UNIX time beyond which the paycode should not be used. 0 for no expiry. Use 0 for versions 1,2,3,4. |
| 5 | checksum | char | checksum calculated the same way as [Cashaddr](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/cashaddr.md#bch). |

For a recipient who intends to receive to a p2sh-multisig addresses, encode the following in base32 using the same character set as cashaddr:

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | version | uint8 | paycode version byte; 3 and 4 for p2sh-multisig (mainnet), 7 and 8 for p2sh-multisig (testnet), among them 4 and 8 to force offline-communication only. |
| 1 | prefix_size | uint8 | length of the filtering prefix desired, 0, 4, 8, 12 or 16 bits for versions 1 through 8 ; 0 if no-filter for full-node or offline-communications. If used, recommend >= 8. |
| 4 bits | multisig_setup_m | uint4 | instruction on constructing the multisig m-of-n to be paid to. m parties who can recover funds. m > 1, m <= n |
| 4 bits | multisig_setup_n | uint4 | instruction on constructing the multisig m-of-n to be paid to. n parties total. n > 1, m <= n |
| 33 | scan_pubkey | char | 256-bit compressed ECDSA/Schnorr public key of the recipient used to derive common secret |
| 33 | spend_pubkey1 | char | First compressed ECDSA/Schnorr public key of the recipients |
| 33 | spend_pubkey2 | char | Second compressed ECDSA/Schnorr public key of the recipients |
| ... | ... | ... | ... |
| 33 | spend_pubkeyn | char | nth compressed ECDSA/Schnorr public key of the recipients |
| 4 | expiry | uint32 | UNIX time beyond which the paycode should not be used. 0 for no expiry. Use 0 for versions 1,2,3,4. |
| 5 | checksum | char | checksum calculated the same way as [Cashaddr](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/cashaddr.md#bch). |

The payment code shall be prefixed with `paycode:`, and can be optionally suffixed with offchain communications networks it supports in URI, e.g. `?xmpp=johndoe@something.org&matrix=@john123:something.com`. If no additional suffix is detected, the default offchain relay method, a necessity for version 2, 4, 6 and 8, is Ephemeral Relay service (see below).

## Private key format

For the easy facilitation of paper wallets and inter-wallet transfers, the scan private key shall begin with "rpriv", the spend pubkey "spriv", followed each by these fields encoded in base32 using the same character set as cashaddr:

| Field Size | Description | Data Type  | Comments |
| -----------|:-----------:| ----------:|---------:|
| 1 | version | uint8 | paycode version byte |
| 1 | multisig_setup | uint4 + uint4 | instruction on constructing multig m-of-n (see above). 0 on both if P2PKH |
| 1 | prefix_size | uint8 | length of the filtering prefix desired, 0, 4, 8, 12 or 16 bits for versions 1 through 8 ; 0 if no-filter for full-node or offline-communications. If used, recommend >= 8. |
| 33 | privkey | char | 256-bit ECDSA/Schnorr private key |
| 5 | checksum | char | checksum calculated the same way as [Cashaddr](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/cashaddr.md#bch). |

## Paycode creation from two keypairs (P2PKH)

Obtain two ECDSA/Schnorr keypairs from a wallet, and designate one as the "scanning" pubkey and the other as the "spending" pubkey.

Any common-secret-derived keypairs detected from incoming payments are additionally stored in the wallet file for history and spending, to minimize future bandwidth and CPU usage. Historical receive addresses do not need to be continuously monitored after first receipt. The expiry date should be set to 0 to remain disabled.

**Paycode creation (P2SH-multisig)**

The multisig m-of-n parties do not keep transaction scanning privacy from each other, and must agree on a common scanning keypair. They can subsequently submit one ECDSA/Schnorr spending pubkey each, and set up the paycode using the n+1 public keys. The expiry date should be set to 0 to remain disabled.

## Generating a transaction to payment code (P2PKH)

Sender's wallet shall first check the expiry time embedded in the paycode is at least one week ahead of local clock (skip if expiry time is 0). If expiry time is more than a week ahead, proceed.

Sender's wallet shall determine the outpoints she wants to spend from, and determine which input's public key  is to be used for deriving common secret. For a multi-input transaction whether from single or multiple senders, that designated input should be randomly determined, but limited to within the first 30 inputs. Note that multiple recipients can be included in a single transaction, and each recipient address are free to either be derived from public keys in different designated inputs or the same input.

In the case the designated input is P2PKH, P below is simply its embedded public key. In the case of designating P2SH-multisig, it is the first public key with a valid signature within that input.

A common secret c can be derived as follows:

Q = scan_pubkey

d = scan_privkey

R = spend_pubkey

f = spend_privkey

Q = dG

R = fG

P = eG = first public key embedded in designated input with a valid signature

e = private key paired with public key P

s = integer derived from outpoint spent by designated input

The common secret c = H(H(e · Q) + s) = H(H(d · P) + s). Here, (·) is the multiplication of points over the secp256k1 elliptic curve, and (+ s) is normal arithmetic as part of this derivation function where H() is SHA-256.

Pay to addresses derived from public keys R'<sub>i</sub> = CKDpub(R,c,i), where CKDpub(K,C,i) is the public parent key -> public child key [derivation](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#Public_parent_key_rarr_public_child_key) from BIP32, with the i<sup>th</sup> public key being the i<sup>th</sup> K, starting from i = 0. Addresses should always be generated from compressed pubkeys.

To recap, we use the first pubkey with a valid signature of the transaction's designated input, together with the scan key, to derive a shared secret via Elliptic-curve Diffie–Hellman [(reference)](https://en.bitcoin.it/wiki/ECDH_address). This shared secret is combined with the outpoint to obtain a unique scalar value for this payment that is used to tweak the spend key into unique ephemeral keys that is then used to derive addresses.

Grinding the prefix is accomplished by using different nonces to sign the designated input until the first prefix_size bits of the double-sha256 of the designated input are shared with the scan_pubkey, excluding the low-entropy first byte (skip if prefix_size = 0). "Input" is a combination of the outpoint and scriptsig. The payment transaction is then constructed and ready to be relayed.

Since bitcoin transactions do not have explicit nonces (unlike blockheaders), the nonce in this case is the random integer "k" value used in creating the transaction signature.  Wallets that already use random "k" can simply keep re-selecting random values as the grinding process.

Repeatedly invoking random number generators in a hot wallet may not be desirable for many; for wallets that have implemented RFC6979 for deterministic signatures, the target paycode and nonce can be concatenated to the message (normally the transaction components) that is passed into the function via `ndata`, thus grinding for k for desired signatures while retaining determinism - the same message, private key and desired paycode will always produce the same signature.

For this purpose, we recommend that `ndata` be SHA256 hash of (version||paycode||nonce), where `version` is a 1 byte field and defaults to 1, `paycode` is the entirety of the target payment code including checksum, and `nonce` is a 32-byte field incremented starting at 0 for the grind.

## Generating a transaction to payment code (P2SH-Multisig)

In the case of paying from P2PKH, P below is also the public key of the designated input. In the case of paying from P2SH-multisig, it is the first public key with a valid signature.

The different part is all of the spending public keys will have to be involved in constructing a new P2SH address, described below in paying to a two-of-three P2SH-multisig setup:

Common secret c can be derived as follows:

Q = scan_pubkey

d = scan_privkey

R1, R2, R3 = spend_pubkey

f1, f2, f3 = spend_privkey

Q = dG

R1 = f1G, ...

P = eG = first public key embedded in designated input with a valid signature

s = integer derived from outpoint spent by first input

Common secret c = H(H(eQ) + s) = H(H(dP) + s)

Pay to new P2SH addresses constructed from keys R1'<sub>i</sub> = CKDpub(R1,c,i), R2'<sub>i</sub> = CKDpub(R2,c,i) and R3'<sub>i</sub> = CKDpub(R3,c,i), with m of n specified in OP_CHECKMULTISIG script. Addresses should always be generated from compressed pubkeys.

Like the case of sending to P2PKH, the sender uses different nonces to sign the designated input until the first prefix_size bits of the double-sha256 of the designated input are shared with the scan_pubkey, excluding the low entropy first byte (skip if prefix_size = 0). The payment transaction is then constructed and ready to be relayed.

## Relaying: Infrastructure needed

There are two methods of receiving: ***Offchain communications***, which saves on bandwidth but entrusts privacy to relay and retention servers, and ***onchain direct sending***, which is trustless on privacy but requires more bandwidth. We describe a single type of server required for both types, as well as two additional types required to make use of the offchain communications method:

1. ***Recovery servers***: In case of onchain direct sending, only this type of server is needed. The server shall index all transactions that have P2PKH or P2SH-multisig inputs by the last bytes of the double-sha256 at all qualifying inputs, and return all transactions matching requested input double-sha256 prefixes to a client. Required for security anyway in the case of offchain communications. See recovery_server.md for specifications.

2. ***Ephemeral Relay servers***: (Optional) This type of server can be freely signed up for using only public keys as identity, authenticating using signed messages as seen in CashID, and can employ basic rate controls against clients as seen in CashShuffle servers and remain largely permissionless. Simplistically relays encrypted messages from one pubkey identity to another. We can start with one central relay server, and gradually expand to multiple federated servers that share communications - a discovery mechanism should be in place similar to other decentralized applications. Does not store information for any significant lengths of time, stateless, and mostly consumes only bandwidth. See relay_server.md for specifications.

3. ***Retention servers***: (Optional) This type of server retains transaction information for offline clients, and can be permissioned while allowing significant innovation and profit models at scale. Entrusted with client scan keys, retention servers connect to relay servers for their clients, then retrieve, decrypt and broadcast transactions for them. They are also responsible for retaining txid information for easy retrieval when client reconnects. Operators of Retention Servers can be expected to also operate their own Ephemeral Relays federated with other operators. An example that takes advantage of CashAccounts can be found at cashaccounts_retention.md.

## Sending: Onchain direct sending

After a transaction is generated, if the sending wallet detects the version allows onchain direct sending, it can simply broadcast the transaction to the Bitcoin Cash network and let it be mined. No notification to the recipient is needed.

## Receiving: Onchain direct

If onchain direct sending is used, receiving is relatively straightforward. The recipient shall connect to a Recovery Server and attempt to download all transactions where at least one of the inputs has double-sha256 that match his payment code's prefix, derived from prefix_length bits of his scanpubkey excluding the first low-entropy byte, since he was last online. This will cost bandwidth that is approximately 1/256 of downloading the full blockchain (in the case prefix length = 8 bits; recovery servers may choose to deny excessively short prefix lengths), and less if longer prefix length is specified.

Upon receiving subscribed transactions, the wallet can then attempt, for each input where double-sha256 prefix matches its paycode and is one of the qualifying type (P2PKH or P2SH-multisig), to derive common secret c from scan_privkey, outpoint spent by that input and the first public key embedded in each input with a matching double-sha256 prefix. If no output addresses match the address generated from R'<sub>0</sub> = CKDpub(R,cG,0), move on to the next matching input within limit of the first 30 inputs; if no inputs are left in the transaction, discard the transaction. If an addresses R'<sub>0</sub> is found, another address R'<sub>1</sub> is derived from that input and attempted to match available outputs, until no more addresses can be found for a given i. This step can also be performed by specialized, trusted servers entrusted with scan privkeys.

Upon obtaining transactions filtered by the scan_privkey, the receiving wallet then stores it locally and assign a spending keypairs R'<sub>i</sub> and h<sub>i</sub> = [CKDpriv](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#Private_parent_key_rarr_private_child_key)(f,c,i) to it. Funds are now available to be spent.

## Sending: Offchain communications

(Optional) If the paycode specifies offchain communications via setting the version byte and does not specify additional relay methods, the sending wallet shall attempt to relay through Ephemeral Relay servers. The constructed transaction shall first be encrypted with the payment code's scan pubkey, using a common ECDSA-based scheme such as [electrum-ECIES](https://github.com/Electron-Cash/Electron-Cash/blob/master/lib/bitcoin.py#L690), then handed off to a relay server. The transaction is considered "Sent" when the sending wallet detects the same transaction being broadcasted by a retention server.

To remain trustless against the possibility of relay or retention servers denying service, after a short timeout (e.g. 30 seconds), if the transaction is not detected, the sending wallet shall consider the broadcast failed and construct a "clawback" transaction that spends the same output to a new address she controls. This is to avoid the case where the trade is voided - recipient never sends goods or services to the sender - yet after some time the recipient broadcasts the transaction anyway, robbing the sender.

## Receiving: Offchain communications

(Optional) A Retention Server, subscribing to Relays using a client's scan privkey, receives the encrypted transaction, then decrypts and broadcasts to the Bitcoin Cash network; invalid transactions can be discarded. The server then proceeds to store the relevant txids calculated from the broadcasted transaction for its clients until retrieved - retention time can vary depending on provider, from several weeks to indefinite depending on specific quality of service desired.

Clients can log onto retention servers and retrieve their incoming txids. Depending on the level of service, the login method can be relatively permissionless (depending on CashAccounts, for example; see cashaccounts_retention.md) or permissioned in premium or other methods. Specific arrangements depend on the exact setup and can even be extended to use other communications protocols such as XMPP and Tox, to be determined by the implementing wallet.

After a client fetches a txid from the retention server, he should proceed to request the full transaction from a node, and extract spending keypair as described above. Then the transaction and spending keypair for its output should be stored locally.

Depending on the specific setup, if a client suspects either server downtime, malicious denial of service, or expired retention, it shall connect to a Recovery Server and attempt to recover funds as described in Onchain Direct transactions.

**Receiving to P2SH-Multisig**

The scanning and filtering part shall work exactly like in P2PKH. Once the multisig parties have a filtered transaction ascertained by the scan_privkey, the receiving parties can then each assign public keys R1'<sub>i</sub>, R2'<sub>i</sub>... and private keys CKDpriv(f1,c,i), CKDpriv(f2,c,i)...to the i<sup>th</sup> outputs within the limit of 30. With these keysets, the address can be spent from normally.

## Expiration time

Note: Expiry date is not expected to be relevant in the near term, so it's recommended that receiving wallets set it to zero when generating. Sending wallets are still recommended to implement it to remain compatible with possible future scalability and usability changes.

Whether it uses onchain direct sending or offchain communications, as long as the wallet remains compatible with seed recovery via recovery servers, it will require a fixed fraction of total Bitcoin Cash bandwidth for that purpose. While the consumption does not have to be latency sensitive in the context of a light wallet, it is vulnerable to long term total traffic fluctuation. If the prefix is too short while network traffic gets much higher, the wallet might have difficulty running as bandwidth requirement rises with network traffic. On the other hand, if the prefix is too long when network traffic is low, the wallet's privacy is degraded as its anonymity set shrinks, in addition to creating undue burden on sending wallets for grinding.

In order to mitigate this, an optional expiry time can be added; sending wallets shall respect the expiry time by yielding an error if attempting to send past it - any funds that are sent overriding the expiry is considered lost, and the recipient has done due diligence warning the sender in the paycode.

When scanning, nodes or wallets can allow for a certain amount of buffer beyond the expiry date, in case a transaction is sent before but remains unconfirmed beyond the expiry - in the context of BCH, a week might be more than sufficient.

In addition to addressing scaling concerns, expiration also addresses another usecase complaint - that wallets once established will have to monitor addresses indefinitely, and that there is no clear indicator to a sender whether an address remains usable or not. Such a clear guide embedded in the address itself can serve these cases well and provide unambiguous dates beyond which recipient will be free from the burden of maintaining monitoring and keys.

## Considerations

***Malleability considerations*** While using input hash as the filtering mechanism has advantage in both flexibility and implementation simplicity, input hashes are third-party malleable if colluded with a miner via nonstandard transactions, via exploiting vulnerabilities fixed in Bitcoin Cash's scheduled November 2019 upgrade (MINIMALDATA for P2PKH and NULLDUMMY for P2SH). Even before the fix, these DoS vectors - note that an attacker cannot steal funds - are mitigated by the fact that an attacker cannot easily pinpoint any given recipient's transaction.

***Limits of party combination*** The design allows multiparty inputs and multiparty payouts, with each recipient party capable of deriving multiple addresses for maximum privacy. However, the number of recipient parties must never exceed the number of inputs nor the 30-input limit in any given transaction; i.e. if you intend to pay to three independent parties, you must provide the transaction with at least three inputs, so each can provide for a filter prefix and public key for one of your recipients.

***Anonymity set*** Anonymity set for prefix_size 0 is effectively all transactions with p2pkh outputs (TBD p2sh-multisig); with prefix_size > 0, it is reduced by a factor of 1/2^(prefix_size) at each step for the simplest case where all transactions have only one input. The set may be larger where transactions contain more inputs, up to ~ 700/2^(prefix_size) in the worst case; for most usecases, though, wallets .

***Upper limit of scalability at recovery*** At very large blocksizes, the maximum prefix length possible by the spec is 2 bytes, or about a 1/65536 filter; for a full 128MB sized block, this will mean the client needs to examine about 281kB of data per day of recovery in the minimum case, more for average number of inputs per transaction above 1 - up to about 8.5MB/day in the worst case where the chain is entirely filled with ~4kB, 30-input consolidations. Unless the disparity between client and server technologies change radically, this should be adequate for the forseeable future.

Note that for current versions the prefix length is limited to 16 bits, or 1/65536, which should be comfortable as described. As the chain grows even bigger, the prefix can be made longer to accomodate more filtering, at a cost of more computing power needed from senders.

***DoS via multiple inputs*** Since one transaction can map to multiple prefixes via its multiple inputs, it would seem possible to increase download burden for onchain direct users, as well as offchain users recovering from seed, by posting very large transactions with many inputs each with different prefixes. However, such a scheme will at most be able to amplify the attack of a 100kB transaction against 30 prefixes, which is likely insufficient to be a serious concern for most situations.

***Compatibility with coinjoin-based technologies*** While incoming addresses are only determined upon sending, coinjoin technologies such as Cashshuffle are quite agnostic to how receiving addresses are generated. The more important aspects of these technologies are that 1) addresses are not reused and 2) a ready list of change addresses can be generated from the same seed or master private key, both of which are compatible. Incoming coins can be marked for joins as they are received or recovered just as they can on HD addresses.
