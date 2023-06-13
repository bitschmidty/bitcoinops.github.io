We started off our series stating that many of Bitcoin's privacy and
censorship-resistance stem from the decentralized nature of the network. Users
each running their own nodes reduces central points of potential failure,
surveillance, or censorship. It follows that one primary design goal for Bitcoin
node software is high accessibility of running a node that fully validates
blocks and transactions. By keeping the requirements for running a node low
(memory, computational resources, and network bandwidth), we can not only
maintain the accessibility of running a node, but also defend the node against
exploitation.

A node on the Bitcoin network is a computer with internet connections to unknown
entities on the internet who may launch a Denial of Service (DoS) attack by
crafting messages that cause the node to run out of memory and crash, or spend
its computational resources and bandwidth on meaningless data instead of
accepting new blocks. As these entities are anonymous by design, nodes cannot
predetermine whether a peer will be honest or malicious before connecting, and
cannot effectively ban them if an attack is observed. Thus, it is not just an
ideal to limit the cost of running a full node, but an imperative.

Many DoS protections are built into node implementations to generally prevent
resource exhaustion. For example, if a Bitcoin Core node receives many messages
from a single peer, it only processes the first one and adds the rest to a work
queue to be processed after other peers' messages. A node also typically first
downloads a block header and verifies its Proof of Work (PoW) prior to
downloading and validating the rest of the block. Thus, any attacker wishing to
use block relay to exhaust this node's resources must first spend a
disproportionately high amount of their own resources computing a valid PoW
while the cost to verify the PoW is trivial. It’s important to note this
property does not extend to _unconfirmed_ transaction relay.

General DoS protections are insufficient to expose a node’s consensus engine to
input from the peer-to-peer network. An attacker attempting to [craft a
maximally computationally-intensive][max cpu tx], consensus-valid transaction
may send something like a 1MB [“megatransaction”][megatx mempool space] included
in block #364292 which took an abnormally long time to validate due to
[signature verification and quadratic sighashing][rusty megatx]. An attacker may
also make all but the last signature valid, causing the node to spend minutes on
this transaction, only to find that it is garbage. During that time, a new block
may have arrived but the node was too busy to process it. One can imagine this
type of attack being targeted at competing miners to gain a “head start” on the
next block.

As such, Bitcoin Core nodes impose a maximum standard size and a maximum number
of signature operations (or "sigops") on each transaction, more restricting than
the block consensus limit. While this means some legitimate transactions may not
be accepted or relayed, those transactions are expected to be rare. These are
examples of _transaction relay policy_, a set of validation rules in addition to
consensus which nodes apply to unconfirmed transactions.

Some policies apply to transactions in relation to one another. Bitcoin Core
nodes enforce restrictions on both ancestor and descendant packages. These two
package limits [restrict the computational complexity][se descendant limits] of
mempool insertion and deletion which require updating a transaction’s ancestor
and descendant sets.

The Replace by Fee [policy implemented by Bitcoin Core][bitcoin core rbf docs]
requires that the replacement transaction pay a higher feerate than the
transaction(s) it directly conflicts with, but also requires that it pay a
higher total fee. This fee policy prevents repeated replacements of the same
transaction in which each transaction only pays a small amount more than its
predecessor (e.g. 1 satoshi). In other words, regardless of the feerates of the
original and replacement transactions, the new transaction must pay "new" fees
to cover the cost of its own bandwidth at 1sat/vB.

This incremental relay feerate is also used to [raise][pr 6722] a node's mempool
minimum feerate ("mempoolminfee") when it reaches capacity and evicts
transactions in descendant score order. This dynamic mempool minimum feerate
does not necessarily affect the outcome, as a transaction with a feerate below
the just-evicted transaction would be evicted immediately after being accepted.
However, rejecting the transaction faster affords worthwhile savings in
validation effort.

By default, Bitcoin Core nodes do not accept any transactions below the
"minrelaytxfee" feerate, which sets a “price” for network transaction relay.
Bitcoin Cores also does not verify any signatures until after checking the
feerate requirements. These fees represent a cost to the attacker. Somebody may
send an extremely high amount of transactions that require validation and relay,
but will at some point run out of money to pay the fees.

Nodes apply transaction relay policies in addition to consensus rules when
validating unconfirmed transactions to protect against attacks. However, as
policy is not consensus, two peers may have different policies but still agree
on what the current chain state is. Next week’s post will discuss policy as an
individual choice.

[max cpu tx]: https://bitcointalk.org/?topic=140078
[megatx mempool space]: https://mempool.space/tx/bb41a757f405890fb0f5856228e23b715702d714d59bf2b1feb70d8b2b4e3e08
[rusty megatx]: https://rusty.ozlabs.org/?p=522
[bitcoin core rbf docs]: https://github.com/bitcoin/bitcoin/blob/v25.0/doc/policy/mempool-replacements.md
[pr 6722]: https://github.com/bitcoin/bitcoin/pull/6722
[se descendant limits]: https://bitcoin.stackexchange.com/questions/118160/whats-the-governing-motivation-for-the-descendent-size-limit
