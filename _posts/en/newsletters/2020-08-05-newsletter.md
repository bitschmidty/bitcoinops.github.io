---
title: 'Bitcoin Optech Newsletter #109'
permalink: /en/newsletters/2020/08/05/
name: 2020-08-05-newsletter
slug: 2020-08-05-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter contains our regular sections with recently
transcribed talks and conversations, releases and release candidates,
and notable changes to popular Bitcoin infrastructure projects.

## Action items

*None this week.*

## News

*No significant news about Bitcoin infrastructure development this
week.*

## Recently transcribed talks and conversations

*[Bitcoin Transcripts][] is the home for transcripts of technical
Bitcoin presentations and discussions. In this monthly feature, we
highlight a selection of the transcripts from the previous month.*

FIXME:michaelfolkson

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

FIXME:harding to update Tuesday

- [C-Lightning 0.9.0][C-Lightning 0.9.0] is the newest major version of
  C-Lightning.  It adds support for the updated `pay` command and new
  `keysend` RPC described in [Newsletter #107][news107 notable].  It
  also includes several other notable changes and multiple bug fixes.

- [Bitcoin Core 0.20.1][Bitcoin Core 0.20.1] is a new maintenance
  release.  In addition to bug fixes and some RPC behavior changes
  resulting from those fixes, the planned release provides compatibility
  with recent versions of [HWI][topic HWI] and its support for hardware
  wallet firmware released for the [fee overpayment attack][].

- [LND 0.11.0-beta.rc1][LND 0.11.0-beta] is the release candidate for a
  new major release.  <!-- FIXME:harding to update Tuesday (LND haven't
  yet published their draft release notes) -->

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1 repo],
[Hardware Wallet Interface (HWI)][hwi], [Bitcoin Improvement Proposals
(BIPs)][bips repo], and [Lightning BOLTs][bolts repo].*

- [Bitcoin Core #19569][] allows Bitcoin Core to fetch the parents of _orphan_
  transactions from peers that relay transactions using wtxid. An orphan
  transaction is an unconfirmed transaction that we receive from a peer where we
  don't yet have the parent transactions, either as part of our best block
  chain, or in the mempool. More precisely, an orphan transaction has at least
  one transaction input whose associated output is not in the UTXO set or our
  mempool's outpoint map.

    When we receive an orphan transaction, we place it in a temporary data
    structure called the orphan set. We then ask the peer that sent us the
    orphan to also send us the parent transactions that we don't yet have. We can
    do that because the orphan refers to the txids of the parent transactions. We
    simply send a `GETDATA` message containing those txids to the peer to
    request the parent transactions.

    For [wtxid relay peers][news108 wtxid relay], transactions are announced
    and requested using the _wtxid_ of the transaction, not the _txid_. However,
    orphan transactions only refer to their parents' txids, not wtxids, so it's
    not possible to request the parent transaction using wtxid. [PR
    #18044][Bitcoin Core #18044], which introduced wtxid relay peers and was
    merged last week, did not permit fetching parent transactions from wtxid
    peers. This follow-up PR allows us to fetch those parents using the txid.

    Fetching parent transactions using txid may eventually be replaced
    by a [package relay][topic package relay] mechanism, where we can
    ask a peer for all the unconfirmed ancestors of a transaction directly.

- [Eclair #1491][] adds partial support for creating, using, and closing
  channels that use [anchor outputs][topic anchor outputs] to both reduce
  onchain transaction fees in normal cases and increase fees when
  necessary for security.  This implementation handles all the basics
  but does not yet support mutual channel closes or actual fee bumping;
  those will be added later.  Additionally, interoperability testing
  with LND's implementation revealed a case when the
  [specification][anchor spec discuss] should be clarified.

- [LND #4488][] updates the minimum CLTV expiry delta users may set to
  18 blocks in line with an [updated recommendation][BOLTs #785].  The
  default remains at 40 blocks.  When there are only this many blocks
  until an LN payment has to be settled, LND will unilaterally close the
  channel to ensure the latest state gets recorded onchain.  However,
  the higher this value is, the more time a payment could become
  temporarily stuck in a channel (either by accident or deliberately).
  This has led some LN implementations to use route finding algorithms
  that optimize for routes with low CLTV expiry deltas, which has in
  turn led some users to set their deltas to values that are especially
  unsafe.  This new minimum should help prevent inexperienced users from
  naively setting an unsafe value.

- [BIPs #948][] BIP174: Clarify that both UTXO types are allowed FIXME:dongcarl

- [BIPs #947][] updates the [BIP325][] specification of [signet][topic
  signet] to change the block signature verification method.  Signets
  are test networks where valid blocks are signed by trusted signers
  rather than using proof-of-work, a change which eliminates some issues
  and makes certain types of testing easier.

    Previously, signet assumed the use of signatures compatible with
    legacy Bitcoin Script (e.g. DER-encoded ECDSA signatures).  After
    this change, signet instead uses a pair of virtual
    transactions---transactions that aren't valid on the block chain and
    aren't included inside the block but which can easily be constructed
    by Bitcoin software (directly or using a [PSBT][topic PSBT]).  The
    first transaction commits to paying the network's trusted signer
    script.  A second virtual transaction then spends the output of the
    first virtual transaction.  The signature or signatures from the
    second virtual transaction are included in the coinbase transaction
    of the block to prove the block is validly signed.

    The main advantage of this new approach is that it allows using
    segwit transactions.   The opcodes available in current segwit v0
    are almost all identical to legacy script, <!-- I think
    OP_CODESEPARATOR is the only change --> so this may seem
    irrelevant---but if segwit v1 ([taproot][topic taproot]) is made
    available on a signet, this will allow signing blocks with [schnorr
    signatures][topic schnorr signatures].  As future protocol changes
    will probably also use segwit, this will allow those features to be
    used as well.  A secondary advantage is that any software or
    hardware that can sign PSBTs for arbitrary inputs will now be able
    to operate as a trusted signer for a signet.

{% include references.md %}
{% include linkers/issues.md issues="19569,1491,4488,948,947,785,18044" %}
[C-Lightning 0.9.0]: https://github.com/ElementsProject/lightning/releases/tag/v0.9.0rc3
[Bitcoin Core 0.20.1]: https://bitcoincore.org/bin/bitcoin-core-0.20.1/
[LND 0.11.0-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.11.0-beta.rc1
[fee overpayment attack]: /en/newsletters/2020/06/10/#fee-overpayment-attack-on-multi-input-segwit-transactions
[news107 notable]: /en/newsletters/2020/07/22/#notable-code-and-documentation-changes
[anchor spec discuss]: https://github.com/lightningnetwork/lightning-rfc/pull/688#issuecomment-661669232
[news108 wtxid relay]: /en/newsletters/2020/07/29/#bitcoin-core-18044
