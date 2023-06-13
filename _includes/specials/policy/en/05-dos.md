We started off our series stating that many of Bitcoin's privacy and
censorship-resistance stem from the decentralized nature of the
network. Users each running their own nodes reduces central points of
potential failure, surveillance, or censorship. It follows that one
primary design goal for Bitcoin node software is high accessibility of
running a node. Requiring each Bitcoin user to purchase expensive
hardware, use a specific operating system, or spend hundreds of
dollars per month in operational costs would very likely reduce the
number of nodes on the network.

Additionally, the role of a full node is wide open to threats. A node
on the Bitcoin network is a computer with internet connections to
unknown entities on the internet who may send data to be processed.
One or more connections may launch a Denial of Service (DoS) attack by
crafting messages that cause the node to run out of memory and crash,
or spend its computational resources and bandwidth on meaningless data
instead of accepting new blocks. As these entities are anonymous by
design, nodes cannot predetermine whether a peer will be honest or
malicious before connecting, and cannot effectively ban them if an
attack is observed. Thus, it is not just an ideal to limit the cost of
running a full node, but an imperative.

Many DoS protections are built into node implementations to generally
prevent resource exhaustion. For example, if a Bitcoin Core node
receives many messages from a single peer, it only processes the first
one and adds the rest to a work queue to be processed after other
peers' messages. A node also typically first downloads a block header
and verifies its Proof of Work prior to downloading and validating the
rest. Thus, any attacker wishing to exhaust this node's resources
through sending confirmed transactions must first spend a
disproportionately high amount of their own resources computing a
valid Proof of Work. The asymmetry between the huge cost for PoW
calculation and the trivial cost of verification denies block relay as
an avenue for DoS attacks. It’s important to note this property does
not extend to _unconfirmed_ transaction relay.

General DoS protections are insufficient to expose a node’s consensus
engine to input from the peer-to-peer network. An attacker attempting
to [craft a maximally computationally-intensive][max cpu tx], consensus-valid
transaction may send something with thousands of non-segwit inputs,
each of which requires the node to hash the signed data (a process
that scales quadratically with the number of inputs) and verify the
signature – both very computationally expensive processes. Block
#364292 included a 1MB [“megatransaction”][megatx mempool space] with
thousands of inputs which took an abnormally long time to validate due
to [signature verification and quadratic sighashing][rusty megatx]. An
attacker may also make all but the last signature valid, causing the
node to spend minutes on this transaction, only to find that it is
garbage. During that time, a new block may have arrived but the node
was too busy to process it. One can imagine this type of attack being
targeted at competing miners to gain a “head start” on the next block.

As such, Bitcoin Core nodes impose a maximum standard size and a
maximum number of signature operations (or "sigops") on each
transaction, more restricting than the block consensus limit. While
this means some legitimate transactions may not be accepted or
relayed, those transactions are expected to be rare. These are
examples of _transaction relay policy_, a set of validation rules in
addition to consensus which nodes apply to unconfirmed transactions.

Some policies apply to transactions in relation to one another. A
previous post discussed an ancestor package limit which also assists
the block template building algorithm. Bitcoin Core nodes enforce
symmetrical restrictions on descendant packages. Like block template
production, the eviction algorithm is more effective when working with
smaller packages. These two package limits [restrict the computational
complexity][se descendant limits] of mempool insertion and deletion
which require updating a transaction’s ancestor and descendant sets.

In the last post, we mentioned the 1sat/vB minimum relay feerate
policy ("minrelaytxfee" in Bitcoin Core), which sets a “price” for
network transaction relay. By default, Bitcoin Core nodes do not
accept any transactions below this feerate. In fact, Bitcoin Core
nodes do not verify any signatures before checking the feerate
requirements. A non-mining node doesn’t ever receive fees – they are
only paid to the miner who confirms the transaction – and some network
bandwidth has already been used by the time the fee is checked.
However, fees represent a cost to the attacker. Somebody may send an
extremely high amount of transactions that require validation and
relay, but will at some point run out of money to pay the fees.

The Replace by Fee [policy implemented by Bitcoin Core][bitcoin core
rbf docs] requires that the replacement transaction pay a higher
feerate than the transaction(s) it directly conflicts with, but also
requires that it pay a higher total fee. The additional fees (the fees
paid by the replacement subtracted by the total fees paid by all of
the transactions it replaces) divided by the replacement transaction's
virtual size must be at least 1sat/vB. This fee policy is not
primarily concerned with the incentive compatibility. Rather, it
prevents repeated replacements of the same transaction in which each
transaction only pays a small amount more than its predecessor (e.g. 1
satoshi). In other words, regardless of the feerates of the original
and replacement transactions, the new transaction must pay "new" fees
to cover the cost of its own bandwidth at 1sat/vB.

This incremental relay feerate is also used to [raise][pr 6722] a
node's mempool minimum feerate ("mempoolminfee") when it reaches
capacity and evicts transactions in descendant score order: the new
mempool minimum feerate is set to 1sat/vB higher than the descendant
score of the last transaction (package) evicted, then decays downwards
towards 1sat/vB at a rate adjusted for how quickly the mempool
empties. Setting a dynamic mempool minimum feerate does not
necessarily affect the outcome, as a transaction with a feerate below
the just-evicted transaction would be evicted immediately after being
accepted.  However, rejecting the transaction faster affords
worthwhile savings in validation effort.

A node that fully validates blocks and transactions requires resources
including memory, computational resources, and network bandwidth. We
must keep the requirements for these resources low in order to
maintain the accessibility of running a node and to defend the node
against exploitation. General DoS protections are not enough, so
nodes apply transaction relay policies in addition to consensus rules
when validating unconfirmed transactions. However, as policy is not
consensus, two peers may have different policies but still agree on
what the current chain state is. Next week’s post will discuss policy
as an individual choice.

[max cpu tx]: https://bitcointalk.org/?topic=140078
[megatx mempool space]: https://mempool.space/tx/bb41a757f405890fb0f5856228e23b715702d714d59bf2b1feb70d8b2b4e3e08
[rusty megatx]: https://rusty.ozlabs.org/?p=522
[bitcoin core rbf docs]: https://github.com/bitcoin/bitcoin/blob/v25.0/doc/policy/mempool-replacements.md
[pr 6722]: https://github.com/bitcoin/bitcoin/pull/6722
[se descendant limits]: https://bitcoin.stackexchange.com/questions/118160/whats-the-governing-motivation-for-the-descendent-size-limit
