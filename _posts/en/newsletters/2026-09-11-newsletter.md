---
title: 'Bitcoin Optech Newsletter #422'
permalink: /en/newsletters/2026/09/11/
name: 2026-09-11-newsletter
slug: 2026-09-11-newsletter
type: newsletter
layout: newsletter
lang: en
---
FIXME:bitschmidty

## News

- **Faster hash tables for txid-keyed data:** Pieter Wuille [posted][wuille
  siphash] to Delving Bitcoin describing SipHash-1-3-UJ, a custom variant of
  the [SipHash][] function that Bitcoin Core 32.0 (expected in October) uses
  for its UTXO cache and other hash tables keyed by txids. Bitcoin Core uses
  hash tables throughout its validation and P2P code to track what peers have
  sent, deduplicate data, and cache UTXOs. Because peers ultimately supply the
  data in those tables, an attacker could craft entries that all land in the
  same bucket, degrading performance. Bitcoin Core defends against this with
  SipHash keyed by a secret salt generated at startup. Wuille notes this is
  overkill for the UTXO cache, whose keys are a txid and output index, because
  the txid is already a cryptographic hash that an attacker can only influence
  by grinding.

  SipHash-1-3-UJ makes three changes. It uses the SipHash-1-3 round counts
  already standard in Python and Rust hash tables, drops byte padding in favor
  of fixed 64-bit input blocks (since all keys are the same size), and adds
  256-bit "jumbo" blocks that consume a whole hash-output-sized input in a
  single round, permitted only when the input is itself a cryptographic hash.
  Together these cut the work per lookup from 14 rounds to 5, or from 17.0 to
  10.6 nanoseconds on Wuille's machine. The construction has not been
  formally analyzed, although SipHash coauthor Jean-Philippe Aumasson reviewed
  it briefly without finding an attack, and Wuille argues that the secret keys
  make it more than sufficient for hash table use. The function was introduced
  in [Bitcoin Core #35215][] (see [Newsletter #415][news415 siphash]), and
  Andrew Toth [noted][toth siphash] that it also speeds up the set used to
  filter same-block spends in the parallel prevout fetching added in
  [Bitcoin Core #35295][] (see [Newsletter #414][news414 prevout]). It is also
  used in the compact `-txindex` format from [Bitcoin Core #35531][] (see
  [Newsletter #419][news419 txindex]). Anthony Towns [asked][towns siphash]
  whether Wuille would specify the function in a BIP, suggesting it could be
  useful in over-the-wire protocols such as block template sharing,
  [erlay][topic erlay], and a future version of [compact block relay][topic
  compact block relay].

- **Babilonia, a coinjoin that is also a covert bet:** Adam Gibson
  [posted][gibson babilonia] to Delving Bitcoin a [paper][babilonia paper] and
  [implementation][babilonia impl] of Babilonia, a two-party protocol in which
  a fair bet is settled inside transactions that look like ordinary payments
  or [payjoins][topic payjoin]. The idea grew out of the earlier Delving
  discussion of emulating an `OP_RAND` opcode (see Newsletters [#340][news340
  rand] and [#341][news341 rand]). Two parties cofund a shared
  [taproot][topic taproot] output, the dealer commits to a secret choice using
  simple sigma-protocol zero-knowledge proofs and [adaptor signatures][topic
  adaptor signatures], and revealing the adaptor secret when the payout is
  cosigned lets the winning player compute the key needed to claim the pot.
  The protocol needs two payment-sized transactions and one confirmation wait,
  with an occasional third transaction, and in the cooperative case is
  indistinguishable from a payment. Gibson describes the goal as a
  steganographic version of [coinjoin][topic coinjoin]: like payjoins, the
  transfer of value breaks subset-sum analysis, and a small "pot sweetener"
  can pay a counterparty to participate without leaving the fee fingerprint
  that JoinMarket-style coordination does. He contrasts it with a more
  powerful construction by Paul Gerhart and coauthors that uses an oblivious
  pseudorandom function to support very small win probabilities such as
  lotteries, but at the cost of requiring a swap structure and a paid
  evaluation step, whereas Babilonia supports only a countable set of
  outcomes but allows a single-transaction join flow.

  Oleksandr Kurbatov [suggested][kurbatov babilonia] reordering the protocol
  so that every proof and signature is exchanged before any funds move,
  removing a window in which both parties have locked coins but the settlement
  is unsigned. Gibson confirmed the implementation already did this and
  updated the paper. Kurbatov also sketched "flip channels" that would settle
  many bets offchain and a way to route bets between parties without a shared
  channel using [PTLCs][topic ptlc]. In a later [update][gibson babilonia
  update], Gibson reported that the paper's privacy claim does not hold as
  stated: because the pot is split according to the agreed odds, the winner's
  payout is a fixed multiple of their contribution, which an analyst can
  detect across the two transactions. He proposes fixing this by aggregating
  several bets with different odds under one funding output so that the net
  payout has an arbitrary ratio, and now leans toward [coinswaps][topic
  coinswap] and LN payments as the most promising applications.

- **Variable difficulty controllers can strand a slowing miner:** Eric Price
  [posted][price vardiff] to Delving Bitcoin an analysis of why some variable
  difficulty (vardiff) controllers used by [mining pools][topic pooled mining]
  fail when a miner's hashrate drops, for example because it is curtailed for
  grid demand response or throttled for heat. A pool sets a per-connection
  share difficulty so that each miner submits a share, a block header with
  proof of work below the network target, at roughly a target rate. Price
  models vardiff as a feedback loop in which the share stream is the sensor
  and difficulty is the actuator, and observes that the sensor fails in the
  worst possible direction: when difficulty is too high for a slowed miner,
  shares become rare, so the controller has little information about the
  miner's actual rate and cannot lower difficulty. A controller that adjusts
  only when shares arrive therefore freezes at a difficulty the miner cannot
  work its way down from, and one that moves in fixed steps may never adjust
  at all. Price argues this is a correctness problem rather than a tuning
  problem, since no gain or step size can recover information that is not
  arriving, and that the fix is for the controller to also ease difficulty on
  a timer when shares stop. He describes a proxy-based test any miner can run
  against their pool and notes the work began as an attempt to replace the
  Stratum v2 reference vardiff.

  Anthony Towns [replied][towns vardiff] that in a Stratum v2 or DATUM
  deployment, miners sit behind a local proxy or gateway that either issues
  the curtailment commands itself or can easily observe them, and that can
  request a difficulty change upstream, making this an engineering problem
  for the proxy rather than a flaw in vardiff. Price [responded][price
  frontier] with a companion post arguing the problem relocates rather than
  disappears: the pool only sees an aggregate of each proxy's miners, so
  per-miner health becomes the proxy's job, and the pool and proxy become two
  linked controllers. Towns [outlined][towns vardiff rules] a simple proxy
  policy, including halving a connection's difficulty after 30 seconds
  without a share and using Stratum v2's `UpdateChannel` message or Stratum
  v1's `mining.suggest_difficulty` to request upstream changes, and noted that
  DATUM avoids negotiation by having the gateway commit its chosen difficulty
  in the coinbase transaction. Price agreed the rules match the controller he
  advocates, adding that forwarding curtailment commands is free within an
  operator's own equipment but becomes an unverifiable request at the
  proxy-to-pool trust boundary.

- **Silent payments light client measurements:** Rob Segers [posted][segers
  sp] to the Bitcoin-Dev mailing list measurements for the [silent
  payments][topic silent payments] light client protocol discussed on Delving
  Bitcoin in 2024 (see [Newsletter #305][news305 sp light]), where the thread
  had stalled waiting for data. Running a BlindBit Oracle index over every
  block since taproot activation, he found the full serving index is about
  109 GB, far larger than the often-quoted 1.7 to 2.8 GB, which counts only
  the 188 million tweaks. Per block, a standard [BIP158][] [compact block
  filter][topic compact block filters] averages 22.6 kB, a taproot-only filter
  averages 3.7 kB (about 6.1 times smaller), and the payload for the current
  oracle protocol, which dropped filters in favor of serving txids, tweaks,
  and output prefixes directly, averages 59.0 kB. He estimates the current
  approach costs about 2.1 times the bandwidth of a filter-based design in
  exchange for zero false positives and no per-match block downloads. Segers
  also reported several places where deployed software has drifted from the
  written specifications, and described publishing per-block commitments to
  the sorted set of tweaks, checkpointed to Nostr, so that a server that
  silently omits a tweak can be held accountable after the fact. He suggests
  agreeing on a canonical per-block tweak set now, since the same set would
  underpin a future P2P-served index or coinbase commitment.

## Releases and release candidates

_New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates._

FIXME:Gustavojfe

## Notable code and documentation changes

_Notable recent changes in [Bitcoin Core][bitcoin core repo], [Core
Lightning][core lightning repo], [Eclair][eclair repo], [LDK][ldk repo],
[LND][lnd repo], [libsecp256k1][libsecp256k1 repo], [Hardware Wallet
Interface (HWI)][hwi repo], [Rust Bitcoin][rust bitcoin repo], [BTCPay
Server][btcpay server repo], [BDK][bdk repo], [Bitcoin Improvement
Proposals (BIPs)][bips repo], [Lightning BOLTs][bolts repo],
[Lightning BLIPs][blips repo], [Bitcoin Inquisition][bitcoin inquisition
repo], and [BINANAs][binana repo]._

FIXME:Gustavojfe

{% include snippets/recap-ad.md when="2026-09-15 16:30" %}
{% include references.md %}
{% include linkers/issues.md v=2 issues="35215,35295,35531" %}
[wuille siphash]: https://delvingbitcoin.org/t/faster-txid-hash-tables-with-siphash-1-3-uj/2834
[toth siphash]: https://delvingbitcoin.org/t/faster-txid-hash-tables-with-siphash-1-3-uj/2834/2
[towns siphash]: https://delvingbitcoin.org/t/faster-txid-hash-tables-with-siphash-1-3-uj/2834/3
[SipHash]: https://en.wikipedia.org/wiki/SipHash
[news415 siphash]: /en/newsletters/2026/07/24/#bitcoin-core-35215
[news414 prevout]: /en/newsletters/2026/07/17/#bitcoin-core-35295
[news419 txindex]: /en/newsletters/2026/08/21/#bitcoin-core-35531
[gibson babilonia]: https://delvingbitcoin.org/t/babilonia-probabilistic-coinjoin-and-covert-betting/2704
[babilonia paper]: https://github.com/AdamISZ/babilonia-paper
[babilonia impl]: https://github.com/AdamISZ/babilonia
[news340 rand]: /en/newsletters/2025/02/07/#emulating-op-rand
[news341 rand]: /en/newsletters/2025/02/14/#continued-discussion-about-probabilistic-payments
[kurbatov babilonia]: https://delvingbitcoin.org/t/babilonia-probabilistic-coinjoin-and-covert-betting/2704/3
[gibson babilonia update]: https://delvingbitcoin.org/t/babilonia-probabilistic-coinjoin-and-covert-betting/2704/6
[price vardiff]: https://delvingbitcoin.org/t/research-a-clockless-vardiff-strands-a-slowing-miner/2718
[towns vardiff]: https://delvingbitcoin.org/t/research-a-clockless-vardiff-strands-a-slowing-miner/2718/4
[price frontier]: https://delvingbitcoin.org/t/vardiff-belongs-at-the-frontier/2734
[towns vardiff rules]: https://delvingbitcoin.org/t/research-a-clockless-vardiff-strands-a-slowing-miner/2718/7
[segers sp]: https://groups.google.com/g/bitcoindev/c/qqDYHnnoM7k
[news305 sp light]: /en/newsletters/2024/05/31/#light-client-protocol-for-silent-payments
