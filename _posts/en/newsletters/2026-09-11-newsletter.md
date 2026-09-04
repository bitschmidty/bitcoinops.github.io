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
  siphash] to Delving Bitcoin a writeup of SipHash-1-3-UJ, the custom
  [SipHash][] variant that Bitcoin Core 32.0 (expected in October) will use for
  its UTXO cache (see [Newsletter #415][news415 siphash]), for the set that
  filters same-block spends during parallel prevout fetching (see [Newsletter
  #414][news414 prevout]), and for the compact `-txindex` format (see
  [Newsletter #419][news419 txindex]), with more uses possible in later
  releases.

  Bitcoin Core hashes the keys of its many internal hash tables with a secretly
  keyed SipHash because peers supply the data in those tables and
  could otherwise craft entries that all land in the same bucket. Wuille
  explains that this is overkill when the key is a txid, which is already a
  cryptographic hash an attacker can only influence by grinding. SipHash-1-3-UJ
  therefore uses the lighter SipHash-1-3 parameters that are the default in
  Python and Rust, drops byte padding since all keys are the same size, and
  adds 256-bit "jumbo" input blocks, permitted only for inputs that are
  themselves cryptographic hashes, that are absorbed in a single internal
  SipRound instead of four. This cuts each lookup from 14 SipRounds to 5, or
  from 17.0 to 10.6 ns on Wuille's machine.

  He notes the construction has not received significant scrutiny from
  cryptographers, although SipHash coauthor Jean-Philippe Aumasson looked at it
  briefly without finding an attack. Anthony Towns [asked][towns siphash]
  whether Wuille would specify the function in a BIP, suggesting it could be
  useful in over-the-wire protocols such as block template sharing,
  [erlay][topic erlay], and a future version of [compact block relay][topic
  compact block relay].

- **Babilonia probabilistic coinjoin protocol:** Adam Gibson
  [posted][gibson babilonia] to Delving Bitcoin a [paper][babilonia paper] and
  [implementation][babilonia impl] of Babilonia, a two-party protocol designed
  to settle a fair bet inside transactions that look like ordinary payments or
  [payjoins][topic payjoin]. The idea grew out of the earlier Delving
  discussion of emulating an `OP_RAND` opcode (see Newsletters [#340][news340
  rand] and [#341][news341 rand]). Two parties cofund a shared [taproot][topic
  taproot] output. The dealer hides a value and the player guesses, with simple
  zero-knowledge proofs and [adaptor signatures][topic adaptor signatures]
  ensuring that a correct guess lets the player derive the key that claims the
  pot. The protocol needs two payment-sized transactions and one confirmation
  wait.

  Gibson describes the goal as a steganographic version of [coinjoin][topic
  coinjoin]: like payjoins, the transfer of value breaks subset-sum analysis,
  and a small "pot sweetener" can pay a counterparty to participate without
  watermarking the transactions the way JoinMarket does. He contrasts it with a
  more powerful construction by Paul Gerhart and coauthors that supports very
  small win probabilities such as lotteries but requires a swap structure and a
  paid evaluation step, whereas Babilonia supports only a small number of
  outcomes but allows a single-transaction join flow. In July he wrote that he
  leans toward [coinswaps][topic coinswap] and LN payments as the most
  promising applications.

  In an August [update][gibson babilonia update], Gibson reported that the
  paper's privacy claim does not hold as stated: because the pot is split
  according to the agreed odds, the winner's payout is a fixed multiple of
  their stake, which an analyst can detect across the two transactions. He
  proposes aggregating several bets with different odds under one funding
  output so that the net payout has an arbitrary ratio. Optech did not analyze
  the protocol in detail.

- **Difficulty control for slowing miners:** Eric Price [posted][price vardiff]
  to Delving Bitcoin an analysis arguing that the variable difficulty (vardiff)
  controllers used by [mining pools][topic pooled mining] can strand a miner
  whose hashrate drops, for example when curtailed for grid demand response.
  Shares, the partial proofs of work a miner submits to show it is hashing, are
  the pool's only signal of a miner's rate. When difficulty is set too high for
  a slowed miner, shares become rare, so a controller that adjusts only when
  shares arrive freezes at a difficulty the miner cannot work its way down
  from. Price argues the fix is to also lower difficulty on a timer when shares
  stop.

  Anthony Towns [replied][towns vardiff] that in Stratum v2 deployments and in
  Ocean's [DATUM][news325 datum] protocol (see [Newsletter #325][news325
  datum]) a local proxy or gateway sits between miners and the pool, can
  observe or directly issue curtailment, and can request an upstream change
  with Stratum v2's `UpdateChannel` message, while DATUM avoids negotiation
  entirely by having the gateway commit its chosen difficulty in the coinbase
  transaction. Price [agreed][price frontier] the proxy is the right place for
  per-miner control but argued the pool still needs its own aggregate
  controller, since it can no longer see individual miners behind a proxy.

- **Silent payments light client measurements:** Rob Segers [posted][segers sp]
  to the Bitcoin-Dev mailing list measurements for the [silent payments][topic
  silent payments] light client protocol discussed on Delving Bitcoin in 2024
  (see [Newsletter #305][news305 sp light]), where the thread had stalled
  waiting for data. Running a BlindBit Oracle index over every block since
  taproot activation, he reports the full serving index is about 109 GB. The
  often-quoted 1.7 to 2.8 GB figure covers tweak storage only, and even the 188
  million tweaks alone total about 6.2 GB.

  Per block, a standard [compact block filter][topic compact block filters] as
  specified in [BIP158][] averages 22.6 kB, a taproot-only filter averages 3.7
  kB (about one-sixth the size), and the payload for the current oracle
  protocol, which dropped filters in favor of serving txids, tweaks, and output
  prefixes directly, averages 59.0 kB. He reports the current approach costs
  about 2.1 times the bandwidth of a filter-based design in exchange for zero
  false positives and no per-match block downloads.

  Segers also reports several places where deployed software has drifted from
  the written specifications, including that the thread's conclusion that
  clients should fetch the full block on a match never made it into the spec
  text. His oracle publishes per-block commitments to the sorted set of tweaks,
  checkpointed to Nostr, so that a server that silently omits a tweak can be
  identified after the fact, although he notes this does not stop a server from
  lying to a targeted client, so clients should still fetch the full block on a
  match. He suggests agreeing on a canonical per-block tweak set now, since the
  same set would underpin a future P2P-served index or coinbase commitment.

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
{% include linkers/issues.md v=2 issues="" %}
[wuille siphash]: https://delvingbitcoin.org/t/faster-txid-hash-tables-with-siphash-1-3-uj/2834
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
[gibson babilonia update]: https://delvingbitcoin.org/t/babilonia-probabilistic-coinjoin-and-covert-betting/2704/6
[price vardiff]: https://delvingbitcoin.org/t/research-a-clockless-vardiff-strands-a-slowing-miner/2718
[towns vardiff]: https://delvingbitcoin.org/t/research-a-clockless-vardiff-strands-a-slowing-miner/2718/4
[price frontier]: https://delvingbitcoin.org/t/vardiff-belongs-at-the-frontier/2734
[news325 datum]: /en/newsletters/2024/10/18/#datum-protocol-announced
[segers sp]: https://groups.google.com/g/bitcoindev/c/qqDYHnnoM7k
[news305 sp light]: /en/newsletters/2024/05/31/#light-client-protocol-for-silent-payments
