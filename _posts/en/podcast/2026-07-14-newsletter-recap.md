---
title: 'Bitcoin Optech Newsletter #413 Recap Podcast'
permalink: /en/podcast/2026/07/14/
reference: /en/newsletters/2026/07/10/
name: 2026-07-14-recap
slug: 2026-07-14-recap
type: podcast
layout: podcast-episode
lang: en
---
Mark "Murch" Erhardt, Gustavo Flores Echaiz, and Mike Schmidt are joined by
Sjors Provoost to discuss [Newsletter #413]({{page.reference}}).

{% include functions/podcast-links.md %}

{% include functions/podcast-player.md url="https://d3ctxlq1ktw2nl.cloudfront.net/staging/2026-6-14/427958894-44100-2-6a5c7b8a03b74.m4a" %}

{% include newsletter-references.md %}

## Transcription

**Mike Schmidt**: Welcome everyone to Bitcoin Optech Newsletter #413 Recap.
Today, we're going to talk about some research into using fountain codes that
would let pruned nodes help sync the chain; we have a couple or Releases,
including Bitcoin Core v31.1 and an LND release; and then, we'll jump into some
Notable code changes.  And then, we'll drill in a little bit on the Mining IPC
interface and some additional methods for Stratum v2.  This week, Murch,
Gustavo, and I are joined by one guest.  We'll let him introduce himself.
Sjors, who are you?

**Sjors Provoost**: Hello, I'm Sjors.  I work on Bitcoin Core with a focus on
making Stratum v2 possible, amongst other things.

_Bitcoin Core #34020_

**Mike Schmidt**: Awesome, thanks for joining us.  We're going to jump out of
order so we can get to Sjors' item sooner, which is down in the Notable code
segment, Bitcoin Core #34020, Mining IPC, fetching mempool transactions by ID.
Sjors, this PR adds two new methods to the Mining IPC interface.  I want to get
into those, but I also think it would be useful to hear from yourself maybe an
update over the last several months about what's been going on in Stratum v2
integration, the Mining interface.  It could be high level, we can get
technical as well, but I know we're going to jump into job declaration.  Maybe
you can kind of give us a lay of the land.  What is the maturity level of the
Mining interface?

**Sjors Provoost**: Yeah, so first, I think it might be useful to explain
briefly the difference between the IPC interface and the RPC interface.  I
don't know if you guys have covered that recently, but Bitcoin Core has had an
RPC interface for decades, well, a decade.  And that's basically mainly sending
commands and then getting a bunch of JSON back.  And for small commands like,
you know, "Give me the latest transactions of my wallet", that's fine.  But if
you're asking for a block template, now you have to take 4 MB of transaction
and put it into a JSON object, and that's not very efficient.  So, for this and
other reasons, the IPC interface uses something called Cap'n Proto over a Unix
socket, and it's a much more efficient way to put data on the wire in a
structured format.  So, that's useful for mining, but it's going to be useful,
I think, for a lot of other tools too.  And I'm especially thinking of indexers
or Electrum server kind of application.  So, I think in Bitcoin Core, we've
been trying IPC for mining specifically, but there are PRs open to expand it
way beyond that.  And in fact, historically, it wasn't designed for mining.
It's just that that's the first use case that came in.  So, that's that
difference.

**Mike Schmidt**: Can you quickly comment on other potential uses of IPC
interfaces?

**Sjors Provoost**: So, historically, the idea was that Bitcoin Core wanted to
split and still I think wants to split the node, which is just a thing that
checks consensus, from the wallet, which is a thing to store your coins but not
everybody uses it, and the GUI, the user interface that's quite user-friendly,
or by 1990 standards.  But it does involve lots of dependencies and again, it's
not all users that want it.  And so, the idea would be if one of these things
has a bug or some problem, it crashes, it doesn't take everything else down.
So, you split both the code, maybe in the same repo, but you split it somehow,
and you make these three processes run as kind of like they're separate
applications where they talk to each other.  And that communication between
these processes needs to be as efficient as possible, and that's kind of why
Cap'n Proto was used.  Yes, that is the back story of IPC, I think, in general.

So, when it came to mining, a little bit of a history, which I think we might
have done in an earlier episode.  But initially, the idea with Stratum v2 was,
let's make Bitcoin Core support that.  And that was kind of the original design
in Stratum v2, was kind of assuming that the Bitcoin Core node would be part of
that ecosystem and it would speak this special language that was designed for
Stratum v2, which includes encryption of certain messages.  And the two, I
would say, advantages of Stratum v2 are it's encrypted and it lets you make
your own block template.  And especially, the making your own block template,
well, who makes a block template?  That would be your own node as a miner.
Some attempts were made in 2023, I think, to add this part to Bitcoin Core
saying, "Hey, we've got Stratum v2.  Why don't you add this to Bitcoin Core and
we're all good?"  For some reason, I ended up taking that over.  And then,
people in Bitcoin Core were like, "Oh, that's a lot of extra code, and it's
opening a network port and it's speaking this new P2P protocol that would need
fuzzers and could have bugs, so maybe don't do that".  And then we kind of came
up with a compromise of saying, okay, Bitcoin Core is not going to speak
Stratum v2 itself, but it will speak this new IPC protocol with messages that
are exactly what Stratum v2 needs.  But then Stratum v2 makes its own little
sidecar app that connects to Bitcoin Core, and also speaks IPC on the other
side.

So, this IPC Mining interface, as we call it, it's not actually a Stratum v2
interface.  It is designed around Stratum v2, but it can be used for DATUM or
anything else, if they wanted to use it to, or really even a legacy-style pool
could be built on this interface.

**Mike Schmidt**: Okay, so similar to what you mentioned earlier, in that the
GUI or wallet could be separated, and there would be functions that would
essentially connect the two.  But it wouldn't be exclusive to necessarily
Bitcoin Core's GUI.  There could be a different GUI calling those same
functions.  Similarly, with the mining interface, the initial use case is
Stratum v2, but other mining protocols could use those functions as well.

**Sjors Provoost**: Yeah, I'd say so.

**Mike Schmidt**: Okay.

**Sjors Provoost**: Now, this was mostly done for the v30 release, the initial
Mining interface design.  And then, after v30, especially the people that work
on Stratum v2, the SRI team, Stratum Reference Implementation, they started
testing it, they found bugs, so we improved things.  V31 that came out a bit
ago should be a lot more stable, so that's good.  And now, there's just a
couple of extra methods that I think would make the integration even better.
And that's kind of what this PR #34020 comes in, unless you want me to explain
something else first?

**Mike Schmidt**: No, I think you can jump into it.

**Sjors Provoost**: Okay, so Stratum v2 has a couple of different roles.  One
role that we just talked about is the template provider.  The template provider
does exactly what you think it does.  It provides templates.  It's kind of a
natural fit for the node.  Then, once you are a miner, you have this template,
now what do you do with it?  You declare it to the pool.  So, there is a role
called the Job Declaration Client, JDC, that talks to, you would be shocked
about the name, a job declarator server, which is run by the pool.  So, the
client talks to the pool.  And so, the pool then, the job declarator server,
needs to actually check whether the thing you're proposing is valid.  Well, it
doesn't need to, but it would be kind of foolish if it just started letting you
mine completely invalid blocks and paying you for your shares.  It's kind of up
to the pool.  And the best way to verify a block is to really just reconstruct
it and then send it to your own node as a pool.

So, what happens is quite similar to a compact block relay.  The miner node
creates a list of txids or wtxids, sends it off to this job declarator server.
It then looks at its own mempool, reconstructs that block, and then any
transaction that's missing, it will ask the miner, "Hey, give me the full
transactions for these because I don't know about them".  Once it reconstructs
the block, it sends it to the Bitcoin node, and then the node says it's good.
There's no PoW, but otherwise it's fine.  And then, the pool can say, "Okay,
dear miner, go ahead and mine on this".  This takes a while, and in Stratum v2,
miners will optimistically start mining anyway, just assuming it'll be
approved.  And then, if it's rejected for some reason, they'll solo mine or
jump to another pool, or whatever.  The problem in this step is, so far, this
is using wtxids, as I mentioned.  And so, the job declarator server does not
actually have a way to get these from the node right now.  Because the job
declarator server is trying to reconstruct a block, it has these wtxids, but it
doesn't actually know the actual transactions, it needs to get those from the
node.  But it can't do that because we only have the getrawtransaction RPC.
And the getrawtransaction RPC takes a txid, not a wtxid.

So, then it was easy to just add another RPC method, but since we're using IPC
anyway for mining, it seemed better to me to just make an IPC method that can
get any transaction by either the txid or the wtxid, and it can get multiple at
the same time.  It uses this efficient wire format.  And so, now with this new
method that just got merged, and should be available in v32, the job declarator
server gets the list of txids, wtxids, and then can ask the node for all the
actual transactions, reconstruct the block, and then send it to the node for
approval.

**Mike Schmidt**: So, previously that was using RPC and now it's using IPC?

**Sjors Provoost**: No, previously it didn't work at all.  So, previously what
they were doing, I think initially what they were doing is they were calling
something like getrawmempool.  They would basically just copy everything out of
the Bitcoin mempool and have their own sort of fake mempool structure, where
they could get transactions.  And then, anything they were missing, they would
ask from the miner over the network and they would add it to their copy of the
mempool.  So, they weren't relying on the node's mempool at all for some
things.  Partially, they were just dumping the whole node mempool; partially,
they were getting missing transactions from miners.  I think they quickly
changed that design because if you have a 2-GB mempool, or something like that,
it gets pretty inefficient to start tracking a copy of all of that.  So, what
they ended up doing now is they create a block template themselves, even though
they're not mining, they're just checking it out.  They create a block template
and that gives them some idea of what the most relevant transactions are.  They
put that in their mempool copy and then they ask whatever's missing to the
miners.

So, it's kind of a workaround.  And with this method, they need less of a
workaround around.  And then, I recently opened a PR that essentially would
make it even less of a workaround.  But we can do that incrementally.

**Mike Schmidt**: Excellent.  It's pretty straightforward, makes sense to me.
Murch, Gustavo, any follow-up questions?  Sjors, what is the best resource for
miners that are curious about Stratum v2?  Like, where would you point them to
learn more about how this works, what we just spoke about, but if they want to
pursue Stratum v2 mining?

**Sjors Provoost**: I would say stratumprotocol.org, or just google Stratum v2.
I think it'll pop up.  It's got the whole spec, but it also has, like, get
started, or what tooling you need to install, etc.  There's been quite some
interesting developments.  I don't know if it's listed on this site, but I
think there's now an Umbrel app as well, where if you have your own node, you
can just install it that way, some of these node-in-the-box installations.  I
think one of the problems with Stratum v2 is that because it started from a
spec with all these different roles, it's quite abstract, so it's a little
difficult to wrap your head around.  But there are now wizards that just tell
you, "Okay, do you want to solo mine or do you want to mine in a pool?  Do you
want to make your own templates if not?"  And then, it gives you all the config
files and kind of makes it work for you.  But I would start at
stratumprotocol.org.

**Mike Schmidt**: Anything else, Sjors, before we wrap up this item?

**Sjors Provoost**: No, that's all I got.

**Mike Schmidt**: Okay, great.  Thanks for joining us today.  You're welcome to
hang on for some other newsletter discussion, or we understand if you have
things you need to do.

**Sjors Provoost**: Yes, thank you.

_Using fountain codes for IBD_

**Mike Schmidt**: We're going to jump back up to the News section.  One item
this week titled, "Using fountain codes for IBD".  This is a news item based on
Lucas's post to Delving Bitcoin.  Lucas was unable to join us, so we're going
to do our best in sort of translating what the idea is here.  Maybe we can
outline a bit about what this research is getting at.  So, obviously you have
this full blockchain, several hundred gigabytes, and keeping a full copy is
something that not every node runner wants to or can do, so many users run
pruned nodes which validate everything but throw away the those old blocks.
Obviously then, a pruned node can't help other nodes sync because it doesn't
have those old blocks to serve.  So, Lucas, who I believe was doing a Vinteum
fellowship, did some research with a professor down in Brazil.  And I think the
original idea was from roasbeef actually, from the Lightning ecosystem, but
also btcd.  And he posted his research on using these things called fountain
codes, to let pruned nodes also contribute full IBD (Initial Block Download) of
other nodes with only a small storage overhead.

So, I think Lucas uses this metaphor of droplets.  And so, you have these
little droplets that would be put onto pruned nodes.  And then, when someone is
wanting to sync doing full IBD, the analogy here is you get all of these
droplets or you connect to many nodes and get many little droplets.  You are
then a bucket node collecting all these droplets, and that if you get enough of
these, you can actually do a full sync, even though all of the different peers
you're connecting to might not have a full copy of the blockchain, which is
interesting.  Maybe I'll pause there.  I know there's some technicals we can
get into with how that works.  Murch, did you get a chance to look at this
proposal?

**Mark Erhardt**: Yeah, I did look at it briefly.  So, my understanding is that
it would require pruned nodes that participate in this service to keep a little
more storage than usual, but they would essentially take a minuscule
cross-section of various block heights.  And then, when you want to sync, you
would re-assemble it from all these cross-sections or droplets that people
serve you.  I think that it got quite a bit of questions and pushback in the
thread.  So, some people were concerned that having a specific set of droplets,
and especially if that's individual, because you randomly pick parts of every
height, would be a perfect fingerprint to re-identify nodes when they change
their IP address.  The other one is that it would be introducing a bunch of
complicated logic to re-assemble and to make sure that you have enough of these
droplets to get them from way more peers than you would usually need to IBD,
and probably would make the IBD slower and more complicated code.

So, the upside would be, of course, that pruned nodes would have a very easy
way of contributing something to IBD.  But if there are nodes that have the
entire blockchain on hand, they would still be much more efficient.  And one
commenter also was concerned that the droplet situation would make it more
difficult if there were some adversarial behavior, where people feed you wrong
droplets and to identify as which ones were incorrect and to ban those peers.
So, I think it's interesting to talk about.  I'm not entirely sold at this
point.

**Mike Schmidt**: Yeah, Murch, you touched on some of the concerns raised
during the discussion.  You need a lot of peers, IBD would be slower, there's
fingerprinting risk, and also potentially increased DoS attack searches.  You
mentioned also potentially malicious nodes.  I think that Lucas outlines the
different nodes in this architecture as there would be the droplet nodes, which
are ones that have validated the chain and they're storing headers and a
handful of droplets for each sort of epoch, as he calls them, which is, I
think, a collection of 1,000 blocks.  So, those would essentially be the pruned
nodes.  There'd be the bucket nodes, the ones that are bootstrapping.  And
then, these potentially malicious nodes would be what he calls murky nodes,
potentially serving corrupted murky droplets to further this droplet analogy.
Yeah, I mean, it's an interesting idea, yet another approach to IBD.  A lot of
those being discussed recently.

Maybe if there's nothing else, we can outline some of the next steps, at least
that Lucas saw, which was it sounds like he's going to be building a proof of
concept in the btcd node implementation, and potentially also then have a
working protocol spec and BIP as next steps as well.  Anything else we missed
on this news item Murch, Gustavo, Sjors?

**Sjors Provoost**: Yeah, so as you pointed out, it slows down IBD compared to
the ideal situation of having nodes that just have the whole blockchain.  But I
guess in the longer run, this kind of stuff will be more useful, because if
there simply aren't that many nodes that have the full blockchain, then the
best thing you've got is not getting the blockchain.  So then, that would be
your comparison.  But I think right now we're nowhere near being short on that.
And in principle, as long as you get the headers from a trusted source, I guess
you can just get the blocks from Amazon, because you know what the hash is.
So, it could be one of those things that's interesting but premature.

**Mark Erhardt**: Pretty much that.  So, if you already have the header chain,
you could BitTorrent the blockchain, or something.  So, while it of course
sucks if you have to have two pieces of software to achieve one thing, I don't
know if currently there's enough demand or need to put this complicated
technology into Bitcoin directly.  Yeah, currently, I think I was going to look
it up on Bitnodes.  We currently have 21,000 nodes that serve the entire
blockchain.  So presumably, that's kind of enough right now.

**Mike Schmidt**: Bitcoin Core developer saying you should get your blocks from
Amazon.  That'll be the headline out of this one.  Thanks, Sjors!

**Sjors Provoost**: You are welcome.

**Mark Erhardt**: Yeah, well, I mean that is the thing.  You have cryptographic
proof via the header chain what exactly the content is in the blockchain, byte
by byte, because the block headers commit to the entire chain content,
especially once you have the coinbase transaction that also commits to the
witnesses.  So, yeah, you still need to validate the data, but where you get
the data, the worst people can do is that they waste your bandwidth, if you do
validate it and run the right software, if you get your Bitcoin node somewhere
strange, then all bets are off, of course.

**Mike Schmidt**: I think we can jump to the Releases and release candidates
segment.  We have two this week and we also have Gustavo back.  Welcome back,
Gustavo.

_Bitcoin Core 31.1_

**Gustavo Flores Echaiz**: Hey guys, thank you for the intro.  Yes, so this
week we have two Releases.  However, ever since we published this newsletter,
two new versions of Bitcoin Core have also been released, which are similar
maintenance releases, which we'll cover from next week.  But let's talk about
v31.1.  This one is a maintenance release that mostly comes from a fix related
to the new -privatebroadcast feature.  So, as we covered in Newsletter #409,
there was a specific bug edge case where if you were using -privatebroadcast
and you had an issue with your v2 transport or v2 P2P protocol transport, also
known as BIP324, it would unfortunately abandon the private connection and
broadcast through clearnet.  So now, the retry could forget the
-privatebroadcast proxy and make a direct IPv4 or IPv6 connection.  So now, if
it fails through v2 transport, it will retry on v1 transport as well, but it
won't forget the proxy override that -private-broadcast sets.  And that's the
main fix, part of 31.1, but there are also other fixes.

Another one that is a huge takeaway is that chainstate database compaction.
So, just Bitcoin Core could randomly start making some chainstate changes.
Yes, Murch?

**Mark Erhardt**: So, we recently introduced that the UTXO database gets
flushed to disk once an hour instead of every 24 hours.  So, during IBD, it'll
flush more often.  Usually, when the UTXO database runs full, it will flush all
the entries to disk, and would only do it every 24 hours if it didn't run full.
So, if your node crashed, you might lose progress on processing the blockchain
of up to 24 hours, where your UTXOs would be dirty and you would have to
reprocess those roughly maybe 140 blocks.  So, we recently introduced, I think
it was in 29, maybe 30, that instead of flushing every 24 hours, we flush every
hour at the latest, but we keep the UTXO database hot.  So, instead of writing
everything to the desk and emptying out the UTXO database, we would keep all
the nondirty UTXOs in the database, unless it was full.  If it's full, we still
flush, but while it's just running, every hour, we would only write the dirty
UTXOs, well, we would write all of them but keep the ones that are still
relevant hot.  So, this means that you don't start overfilling your UTXO
database in memory, and you don't lose as much progress if your node crashes
unexpectedly.

The thing is that previously, so when you write this data to LevelDB, which is
how we store the UTXO set on disk, we would sort of defragment the LevelDB.
And previously, that would happen once per day at most, or even more seldom, I
don't exactly know how often.  But now, with the flush every hour, it would
re-defragment the LevelDB much more often.  And that created a significant
churn on disk, where it would just rewrite large portions of the UTXO set on
disk.  So, that was definitely not intended.  Some people reported that they
saw a lot more disk I/O since this change that was tracked down.  And now, in
31.1 and I think also the upcoming release, or the releases that are out that
we will cover next week, the 30.3 and 29.4 branches also have these backports.
So, the chainstate compaction runs much more seldomly now.  I think it might
have been turned off completely in some cases.  Sorry, I should have read more
into this.

But the main takeaway is that the chainstate compaction, I think, runs once
after IBD and then runs less often, and especially doesn't write the UTXO set
to disk fresh every time it flushes anymore.

**Gustavo Flores Echaiz**: Yes, and I think also it's important to specify that
there is some intentional background compaction that is added in this item.
So, instead of having unpredictable disk activity that was a bit random,
there's now more infrequent, less frequent intentional background compaction,
so your database still gets cleaned up.  And yeah, so these are the main two
items for the v31.1, but it also has other fixes related to wallet migration.
In previous maintenance releases, we covered a bunch of issues related to
wallet migration from legacy to this descriptor wallet.  So, a final small fix
was also included in this release and other small things as well.  You can
check the release notes for more details.

_LND v0.20.2-beta_

Next is LND v20.2-beta.  This is a maintenance release of LND.  So, it has a
few bug fixes.  The first one is something called a DNS fallback panic.  So,
when LND is looking for peers at setup, it will most likely use a DNS seeder
for that.  And it used to, when obtaining DNS records, if a DNS record was
missing an SRV or service record, at first it would consider other types of
records as a service record, and it would accept it to then later fail it,
because it would then realize that it was another type of record, such as an A
record or a CNAME record.  So now, during peer discovery, LND doesn't assume
that every DNS answer is a service record.  It simply validates the DNS record
instead of trusting its type.

There's also an issue related to the interceptor.  So, an LND user can
optionally set up an HTLC (Hash Time Locked Contract) forward interceptor, and
there was also an issue related to that.  I don't know all the details, but if
you were using that, probably worth taking a closer look.  And finally, last
week we covered, in Newsletter #412, that LND had tightened the validation of
the CLTV (CHECKLOCKTIMEVERIFY) expiry of final hop HTLCs.  So now, that is also
part of this maintenance release.

That is it for the Release section.  And now, we move on to the Notable code
and documentation sections.  So, this week, we have three items from Bitcoin
Core and one from three Lightning implementations, Core Lightning (CLN),
Eclair, and LND.  So, in Bitcoin Core, at the beginning of the episode, we
discussed the third item, which is related to the Mining IPC interface.  But
there are two other important items.

_Bitcoin Core #32489_

So, first, #32489 adds a new RPC called exportwatchonlywallet, which it allows
you, through an RPC call, to export a watch-only version of your wallet,
ensuring that it only exports public key material and it doesn't export any
private key material, but also it includes your transactions, your labels, and
all your other metadata.  Previously, you would have to manually export your
public descriptors, and then when you would import them into another Bitcoin
Core node or wallet, you would have to later manually construct such a wallet
by importing those public descriptors.  So, this allows you to easily export a
watch-only version of a descriptor wallet to then use and load onto another
node.  If you'd like to sign with air-gapped hardware wallets with external
keys, you could use that.  Yes, Sjors?

**Sjors Provoost**: Yeah, so there's a tutorial text file inside the Bitcoin
Core repo, called multisig-tutorial.md, or something like that.  And it
basically takes you all the way through how to set up a multisig wallet.  I
think it uses signet, so you can just try it out without losing your coins.
And there's a bunch of really awkward steps in that tutorial where you have to
print all the descriptors and then write a batch script that imports them
again.  It's quite silly.  And every time PRs like this comes in, that tutorial
just gets less awkward.  So, this PR, if you look at it, you'll also see the
tutorial getting easier.  So, I would suggest trying the tutorial.

**Gustavo Flores Echaiz**: Thank you so much for that extra context.

**Mark Erhardt**: Yeah, basically, if you have a wallet with private key
material, there was just no good way of producing the public version of that,
that removes the private key material.  You basically had to export the
descriptor yourself, you had to then strip the private key material.  Or I
think there might have been a flag that you could have the descriptor without
private key material.

**Sjors Provoost**: Yeah, you can render the descriptors without private keys.
It does that by default, it just shows the xpubs.  But you have to then export
the descriptors and then import them again, which is silly.

**Mark Erhardt**: Right.  And I think importdescriptor is fairly new too.

**Sjors Provoost**: Well, if you consider 2019 new.  Importdescriptor is
actually older than the descriptor wallets themselves.

**Mark Erhardt**: Okay, I misremembered.  Thank you for the correction.

_Bitcoin Core #32606_

**Gustavo Flores Echaiz**: Awesome.  We move forward with the next item,
#32606.  So, here, the implementation of the compact block relay protocol is
updated so that Bitcoin Core ignores compact block messages received from peers
in three different scenarios.  So, basically, the compact block relay protocol
works that first, peers have to negotiate with a message, sendcmpct.  And that
message can either make peers agree to send compact blocks unannounced or
announced, right?  So, a peer could request a compact block in the mode 0,
let's say, and in the mode 1, you're basically signaling to a peer that he can
send you compact blocks unannounced.  And basically, what this item does is
that it updates the protocol to first say that you can ignore messages from
peers that have not negotiated support at all; or even if you request it, if
there's no negotiation at all, you should simply ignore those messages.  Or in
the high-bandwidth announcement mode, which is when you send a block without
having a specific request for it, you should ignore the messages if it's sent
to you if you hadn't negotiated that mode, if it's being sent to you in that
way; and then also, when the local node is running blocks-only mode.  Yes,
Murch?

**Mark Erhardt**: Sorry, let me jump in there a little bit.  So, there are two
different ways how we transfer blocks.  One is the old legacy way, where you
send a header or even an inventory message first, and then send the entire
block including transactions.  That hardly gets used anymore by modern Bitcoin
Core nodes.  The alternative way is to send compact blocks, which are sort of a
recipe or a table of contents of what's in the block; basically, a set of
instructions how to reconstruct the block from the mempool.  The idea is that
if your node has been online, you should have seen almost all of the
transactions that made it into the latest block.  And you only need to know
which ones to use in what order in order to reconstruct the block.  So, that
was introduced in 2015, is broadly adopted by now.  And when nodes make a new
P2P connection, they negotiate features.  So, the sendcmpct feature indicates,
well, having a certain version number in the version handshake tells people
whether or not your node should understand the sendcmpct message.  And then,
whether you set 0 or 1 in sendcmpct indicates whether the other peer is chosen
as a low-bandwidth or a high-bandwidth peer.

High-bandwidth peers are explicitly asked to forward the compact block
immediately when they see it, even before validating it; whereas low-bandwidth
peers would only announce a compact block announcement after validating the
block.

**Sjors Provoost**: It's not entirely validating, right?  It's not not
validating.  I think it does validate something, like the PoW in the Merkle
tree, or something.

**Mark Erhardt**: Right.  But it doesn't do the entire transaction validation
and block validation before sending it along.  So, this fast-tracks the block
propagation.  And so, Bitcoin Core's default behavior is to tap three peers as
high-bandwidth peers.  So, these high-bandwidth peers would be like the ones
that sent them blocks early previously, maybe the first ones to announce blocks
to them.  The node will go, "Hey, you seem like a well-connected node.  Please
be my high-bandwidth peer and just send me compact block announcements
immediately when you see them".  And so, what this PR does is, if a node didn't
negotiate compact block filters at all, aka is running an old protocol version,
but then starts sending you compact blocks, do not accept them.  If a node was
not selected as a high-bandwidth peer, but as a low-bandwidth peer and sends
you a compact block without announcing it, also reject them.  And if you're
running in blocks-only mode, reject any compact block announcements, because if
you're in blocks-only, you don't have a mempool.  Without a mempool you can't
reconstruct compact block announcements, so it doesn't make any sense to send
them to you.

The idea, or the motivation here, is that if you send a compact block
announcement, you're sending a table of contents of which transactions are in
the block.  And the peer will ask back for the transactions they're missing.
So, you could attack appear by sending a false compact block announcement that
adds more short IDs for transactions that are actually not in the block, and
then see which of those transactions your peer asks for, to learn which ones
they didn't ask for, because those are the ones in the mempool.  This is
especially dangerous for blocks-only nodes, because blocks-only nodes will only
have their own transactions in the mempool, transactions that they have
announced to peers.  So, you could use this to glean what nodes have in their
mempools.  And in order to make that a little more expensive, we reject compact
block announcements that are coming from peers where we wouldn't expect them.

**Gustavo Flores Echaiz**: Perfectly explained.  Thank you, Murch.

**Sjors Provoost**: Is anyone observing the network to see if such gleaning is
happening?

**Mark Erhardt**: I don't think so.  There was an issue.  So, this PR links to
an issue from, I think it was 2015, maybe originally 2023.  And I think it's
mostly a theoretical issue.  But I guess when we're talking about it, we might
give people ideas.

**Mike Schmidt**: The Bitcoin Network Open Collective, BNOC, maybe some of
those folks are looking at that.

**Mark Erhardt**: Well, hopefully they're not attacking the network, just
monitoring it.

**Sjors Provoost**: I mean I guess there's not that many people that run a
blocks-only node and then also make transactions.  So, it's probably not a big
pond.

**Mark Erhardt**: Hopefully, they would use -privatebroadcast now after 31.1.

**Sjors Provoost**: And not before?

**Mark Erhardt**: Well, it was marked experimental and it had some very
specific circumstances under which it failed.  But well, it's all fixed now.

**Gustavo Flores Echaiz**: Excellent.  Thank you, guys.  Also, want to say that
this is the second improvement to the implementation of Bitcoin Core for
compact block relay.  In last week's newsletter, we also covered an improvement
related to this.

So, the next item is the one we covered at the beginning.  So, two new methods
are added to The Mining IPC interface to request a list of transactions either
by the txid or the wtxid.  So, your node, you as a Stratum v2 client, can
request those transactions from your node.  And this specifically in the case
where a pool has obtained a block template from a miner and it is trying to
obtain the transactions from its node that the miner has included in its block
template.  And this is the first step.  And if the node wouldn't have some
transactions, then the Stratum v2 client could request potentially these
transactions to the miner that sent the block template.

_Core Lightning #9104 and #9292_

Next item is from the CLN repository.  Here are two PRs #9104 and #9292.  Both
of these have implemented the experimental support for the simple_close_option,
which is a new cooperative close protocol.  While relatively new, it was merged
to the BOLTs repository in February 2025.  CLN is the third implementation to
implement this after Eclair and LND.  And briefly, the simple-close protocol is
different in the sense that instead of both peers having to agree on a specific
closing transaction and having to negotiate the fee, which could lead to the
closed transaction to get stuck if disagreement would happen, here instead the
simple close avoids this issue by having both peers propose valid closing
transactions, that both sign for the other, and both of these peers broadcast
these transactions independently.  So, they don't have to agree to a specific
fee.  Both just propose their transactions, the other signs it, they broadcast,
and whichever confirms first is the transaction that closes this channel.

So, the second PR, #9292, fixes an edge case where CLN, in a situation an
output would be uneconomical to create, a peer could decide to simply create a
zero-value OP_RETURN output.  The edge case that was fixed in this second PR is
that CLN could at first accept the OP_RETURN transaction, but then reject it,
which could cause a force close.  So, these two PRs together combined now
represent the experimental implementation for the simple-close option, which is
a new type of cooperative close transactions.

_Eclair #3323_

The next item from the Eclair repository is an extension of something that
Eclair had already implemented for outgoing HTLCs, and it's now being
implemented for incoming HTLCs.  Here, the rule is simple.  If a CLTV expiry is
more than 2,016 blocks, which is about two weeks, then it would be failed.  It
is not initially rejected.  Eclair temporarily accepts the HTLC and then fails
it since rejecting it outright would foreclose the channel.  And here,
initially, the goal is to reduce the risks of funds being locked, that's
specifically for outgoing HTLCs.  But here, Eclair doesn't want incoming HTLCs
with CLTV expiries that are too much in the future to consume HTLC slots
specifically.  So, another type of resource could get consumed from Eclair,
which is why Eclair has this now strict rule by default.

_LND #10832_

And finally, the last item is from the LND repository.  This is a follow-up to
an item we covered in Newsletter #410, which started the groundwork for
implementing BOLT12 offers.  So, here we kind of see the second step to that by
adding support for InvoiceRequest messages.  So, first we saw the Offer message
type in Newsletter #410, we covered that.  And now, we see the InvoiceRequest
message type being added into LND.  Once again, this is just the groundwork, it
doesn't really implement signature verification or checking the invoice request
against the corresponding offer.  That work is left for subsequent PRs, but we
are now seeing the implementation for BOLT12 offers beginning in the LND repo.
And that is the final item, and that completes the section and the whole
newsletter.

**Mike Schmidt**: Great.  Thank you, Gustavo.  And Sjors, thanks for hanging on
and chiming in and for joining us to talk about the Mining interface and
Stratum v2.  Murch, thanks for co-hosting and thank you all for listening.
We'll hear you next week.

**Mark Erhardt**: Bye-bye.

{% include references.md %}
