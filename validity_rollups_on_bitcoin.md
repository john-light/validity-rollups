# ⚠️ DRAFT - POSTED FOR REVIEW ⚠️
> The report will be finalized soon. Until then, please:
> - Be aware that some information may be incorrect
> - Leave a comment if you have any factual / typo corrections or vitally important information to add

# Validity Rollups on Bitcoin

By John Light

Contact the author: https://lightco.in/contact

## Section 0. The history and prehistory of validity rollups

### Section 0.1 Early applications of cryptographic proofs in cryptocurrencies

> This is a very interesting topic.  If a solution was found, a much better, easier, more convenient implementation of Bitcoin would be possible.  
>   
> Originally, a coin can be just a chain of signatures.  With a timestamp service, the old ones could be dropped eventually before there's too much backtrace fan-out, or coins could be kept individually or in denominations.  It's the need to check for the absence of double-spends that requires global knowledge of all transactions.  
>   
> The challenge is, how do you prove that no other spends exist?  It seems a node must know about all transactions to be able to verify that.  If it only knows the hash of the in/outpoints, it can't check the signatures to see if an outpoint has been spent before...
>   
> **It's hard to think of how to apply zero-knowledge-proofs in this case.**
>   
> We're trying to prove the absence of something, which seems to require knowing about all and checking that the something isn't included.
> 
> — Satoshi Nakamoto [1]

It has been over a decade since bitcoin creator Satoshi Nakamoto first considered how zero knowledge (zk) proof technology could be used to improve upon the electronic cash protocol he invented. Originally described by Goldwasser, Micali, and Rackoff in their 1985 paper "The Knowledge Complexity Of Interactive Proof Systems", zk proofs are a type of cryptographic proof defined as "those proofs that contain no additional knowledge other than the correctness of the proposition in question".[2] Over 35 years after that seminal paper, and a decade after Satoshi Nakamoto first discussed using zk proofs in bitcoin, cryptographic proof protocols are now a core component of bleeding-edge blockchain scaling and privacy protocols.

The first concrete proposal for using zk proofs in a bitcoin context came from Greg Maxwell, who wrote about "zero knowledge contingent payments" in the Bitcoin Wiki in 2011. Maxwell later worked with Sean Bowe, Pieter Wuille, and Madars Virza to implement the protocol in 2016.[3, 4] In May 2013, Miers et al published the Zerocoin paper, showing how zk proofs could be integrated directly into the bitcoin protocol to hide the addresses involved in a bitcoin transaction.[5] That same month, Eli Ben-Sasson gave a presentation at the Bitcoin 2013 conference in San Jose, California describing how, in addition to improving privacy, universal (Turing-complete) cryptographic proofs could be used for scaling bitcoin as well.[6] These early proposals foreshadowed and inspired later implementations of crypotographic proofs that have been used to improve the privacy and scalability of blockchains for real users in "mainnet" production environments.

The first cryptocurrency to deploy zk proof technology to mainnet was Firo (fka Zcoin, fka Moneta), an implementation of the Zerocoin protocol that launched in September 2016.[7] This was followed shortly after by the launch of Zcash, the first cryptocurrency to implement zk-SNARK proofs using the Zerocash protocol, in October 2016.[8] The production implementation of zk proofs in Zcoin and Zcash for privacy, and the potential shown in Ben-Sasson et al's work for improving blockchain scalability with cryptographic proofs, led to increased investment and research into improving the technology. This increased investment in turn resulted in significant improvements in the capabilities and performance of cryptographic proofs.[9]

### Section 0.2 The road to validity rollups

At the same time that general research into the applicability of cryptographic proof technology to cryptocurrencies was happening, a parallel track of research was ongoing specifically to improve the scalability of cryptocurrencies. This track of scaling research would eventually converge with the cryptographic proof research.

The fundamental problem scaling researchers needed to solve was that in order to remain free from the dangers of centralization (censorship, double-spending, corruption, etc) it had to be easy for almost any cryptocurrency user to run a "full node" on the network and contribute to ensuring that the rules of the cryptocurrency consensus protocol are followed by block producers, and, if desired or required, become a block producer themselves.[10]

To keep it easy for almost any user to run a full node, the computational effort of verifying the blockchain had to be limited. This put decentralization at odds with scale. As the popularity of cryptocurrencies, especially bitcoin, grew over the years, this tension became increasingly important to resolve. While a wide variety of approaches were proposed and implemented over the years, cryptocurrency developers and researchers mostly focused on three main techniques attempting to resolve the tension between decentralization and scale: onchain optimization, sharding, and offchain transaction execution.

Onchain optimization techniques reduce the amount of resources that are required to process and store transactions that must be executed by full nodes. This can be achieved by decreasing the size in bytes or gas that transactions take up inside each block, and/or by decreasing the computational resources required to verify transactions, for example by optimizing the library used for signature verification. Such optimizations have led to significant improvements in transaction verification and initial block download times in Bitcoin Core.[11] For a blockchain that supports more expressive smart contract languages, "gas optimization" techniques can likewise lead to significant cost savings and throughput increases.[12]

Sharding is a technique where a cryptocurrency's block processing and storage is split into two or more groups of nodes. Security is shared between shards so that no shard is easier to attack than any other shard. The effect of sharding is that full nodes can be confident that all of the cryptocurrency's consensus rules are being followed while only needing to store and execute a fraction of all of the transactions occurring on the network. The more shards there are, the more throughput can be supported without increasing the computational burden on any given full node. Vitalik Buterin first proposed sharding as a means of scaling the Ethereum protocol in a blog post published in October 2014.[13] The first implementation of a sharded blockchain was Zilliqa, which went live in January 2019.[14] Ethereum developers are still planning to implement sharding, although the specifics of how it will work have changed over time.[15]

Offchain transaction execution protocols are designed to take resource load off the "base layer" full node network. When an onchain cryptocurrency transaction is broadcast to the network, full nodes must "execute" the transaction to ensure that it is valid according to the protocol's consensus rules. Exactly how a transaction is executed depends on the specifics of the protocol; for example, some protocols require full nodes to execute scripts to determine the validity of transactions, and some protocols only require full nodes to check the validity of a signature.[16] If all protocol rules are followed, the transaction can be confirmed, and gets relayed to other peers to verify. If any protocol rules are broken, the transaction is dropped.

Offchain transaction execution protocols move transaction execution to a separate, "higher layer" network. Full nodes on the "base layer" then only have to execute the first (deposit) and last (withdrawal or final settlement) transactions in a series of transactions that were executed offchain on the next layer up. The first offchain execution protocol was a version of bitcoin payment channels, first described by Satoshi Nakamoto in a private email to Mike Hearn in 2011, and later implemented by Mike Hearn and Matt Corallo in bitcoinj in June 2013.[17, 18] Today the most popular offchain transaction execution protocol built for bitcoin is the Lightning Network, a decentralized protocol that routes offchain payments through a network of bidirectional payment channels.[19]

In 2017, the popularity of token sales and breedable kittens on Ethereum led to block space congestion that caused gas prices to spike and transaction confirmations to be delayed.[20] Just when Ethereum was having its first taste of "mainstream" adoption, the limits of its design prevented usage of the blockchain from increasing any further. Scalability became a priority, and application and protocol developers began searching for solutions.[21] Short-term solutions that traded off security for throughput, such as sidechains, were quickly implemented.[22, 23] The developer community was not content to settle for sidechains and continued to search for the holy grail: a scaling solution that would enable more usage without either giving up self-custody or harming full node decentralization. While the community expected that Ethereum's eventual sharding upgrade would significantly improve scalability, there was both an urgency to the problem that demanded a more near-term solution and an understanding that even greater scaling gains could be had by combining sharding with other scaling solutions.[24]

One proposed solution that was explored in depth was state channels, a generalization of the payment channel protocols that were first implemented on bitcoin.[25, 26] State channels had several limitations that made them unsuitable as a general scaling solution for all of the different the kinds of applications that were being built on Ethereum: they were difficult to build for multiparty applications where many users are coming and going at different times, they required expensive capital lockups, and they required users to be online to receive updates and file disputes.

Another proposed solution that got a lot of attention was Plasma. First described in a whitepaper published by Vitalik Buterin and Joseph Poon in August 2017, Plasma was a technique for moving transaction execution offchain while still rooting security in the Ethereum "Layer 1" (L1) blockchain.[27] The main improvement over state channels was that, like a sidechain, and unlike state channels, users would not have to lock up lots of liquidity and manage channel balances. However, like state channels, Plasma had a few problems that held it back from being considered the holy grail scaling solution.

Early versions of Plasma had a "mass exit problem" that could lead to L1 congestion and delay withdrawals by weeks in the case of Plasma operator misbehavior.[28] The mass exit problem was a consequence of another problem, the data availability problem.[29] The problem is that in order to be able to unilaterally exit from an L2 protocol, users need to be able to prove that they currently own the assets they are trying to exit with. However, if any piece of data making up the current state of the L2 protocol is missing, even a single bit of data, for example because a block producer has committed the latest block's state root but has not published the block itself, then users do not have all of the data they need to produce the necessary cryptographic proof to exit, and their funds become frozen until the data is made available.

Plasma solved the data availability problem by enabling users to exit with their last known balance if nodes detect that data for the most recent state committed to L1 is unavailable. The cost of this solution was the users had to put some of the plasma chain data on L1 to exit, and if many users have to exit at once, this would require lots of data and transactions on L1, creating the mass exit problem. Later versions of Plasma made mass exits less of a problem, but still required users to be online and verify the plasma chain and withdrawals to monitor for misbehavior.[30] Perhaps the most important problem Plasma had was that it was difficult to add support for Turing-complete smart contracts like those supported on Ethereum L1.[31, 32] This made Plasma unattractive to developers who wanted the flexibility of the EVM.

In 2019 Ethereum developers began thinking about how to solve both the data availability problem and the EVM compatibility problem in ways that also solved the other problems that Plasma and state channels had, such as the liveness requirement. This led developers to revisit older proposals that required users to post a minimal amount of data on L1 for each L2 transaction, in a way combining each of the main three scaling techniques.[33] This category of protocols that put minimal data onchain while keeping transaction execution offchain came to be known as a "rollup" (which got its name from the first implementation of a validity rollup by Barry Whitehat).[34]

Rollups are categorized into two main variants based on the way state transitions were determined to be valid: optimistic rollups, which use fault proofs to enforce correct state transitions, and validity rollups, which use validity proofs to enforce correct state transitions. (Validity rollups are also often called "zk-rollups", but this can be a misnomer since not all validity rollups use zk proofs.)[35] Due to their reliance on validity proofs, validity rollups are considered "trustless" while optimistic rollups require at least one honest party to submit a fault proof if someone attempts to commit an invalid state transition.

With solutions to the most important shortcomings of previous scalability proposals in hand, Ethereum developers began pushing the limits of rollup capabilities. For example, the Fuel and Loopring teams built optimistic and validity rollups, respectively, for p2p payments and atomic swaps, and the Aztec team built a validity rollup for Zerocash-style "shielded" transactions.[36, 37, 38] Eventually Ethereum developers even figured out how to build fully EVM-compatible rollups, first using optimistic rollups and later using validity rollups.[39, 40] (It should be noted that as of the time of writing, all live rollups on Ethereum except Fuel rely on multisigs to secure their bridge contract, which is a different security model than "true" rollups that rely entirely on the consensus of their parent chain for security — Edan Yago has jokingly called these "very optimistic" rollups.[41, 42] The expectation is that as these protocols mature, they will transition to a "true" rollup security model.)

This brings us to the present day. Ethereum developers have redesigned their whole roadmap around rollups, and other blockchains are taking a similar rollup-centric approach.[43, 44, 45, 46] Given the large amount of excitement and investment going into rollup technology, and given the alignment of trustless validity rollups with the trustless ethos of bitcoin, one might wonder: are validity rollups a good fit for bitcoin, too? To answer this question, we must first take a step back and answer the more foundational questions: Is it even technically possible to build validity rollups on bitcoin? And if it is technically possible to build validity rollups on bitcoin, what would be the benefits, costs, and risks associated with doing so?[47]

## Section 1. An introduction to validity rollups

A rollup is a blockchain that stores the state root and at least enough transaction data to recompute the current state from genesis inside of the block of a different "parent" blockchain, while shifting transaction execution "offchain" to a separate node network. This is in contrast to related protocols such as state channels, sidechains, and validia chains, which also execute transactions offchain but keep the vast majority (if not all) of their transaction data offchain as well. The operation of a rollup is otherwise identical to any other other blockchain: transactions are bundled into blocks, those blocks are broadcast to a network for verification, and valid blocks are added to the "tip" of the chain of previous blocks. (And as with other blockchains, if there are multiple valid blocks competing to get added to the chain tip, how to choose which block gets added to the chain tip varies depending on the rollup consensus rules.)

Validity rollups are one of two main rollup variants that have been invented so far.[48] Validity rollups are so-called because they use cryptographic "validity proofs" to ensure that new rollup blocks follow the rules of the rollup protocol. Every time a validity rollup block is created by a rollup block producer, the block producer submits a state update transaction to the parent chain. The rollup state update transaction contains data about each state transition in the rollup block, the new rollup state root after the state transitions in the block are applied, and a validity proof that proves data availability and that the new rollup state root is a valid update to the last valid rollup state root confirmed on the parent chain. Only state update transactions that are accompanied by a valid proof succeed when confirmed State update transactions with an invalid proof will still be confirmed but the transaction will revert and fail (or they will be completely rejected by full nodes and never be confirmed; it depends on the implementation).

The other variant of rollup is the optimistic rollup, which is so-called because rollup withdrawals are "optimistically" considered valid and only proven to be valid or invalid if challenged.[49] Optimistic rollups rely on at least one honest party to watch the rollup smart contract for invalid state updates and submit a challenge transaction if they spot one. To challenge the validity of an optimistic rollup state update, a valid "fault proof" must be confirmed on the parent chain during a challenge window, which usually lasts between several hours and several days. If the fault proof is valid and confirmed within the challenge window, the invalid update will be cancelled, protecting users of the rollup from potential theft. Generally, a bond is posted in the rollup contract upon submission of update and challenge transactions, to prevent griefing attacks. If the owner of the bond misbehaves, for example by submitting either an invalid state update or an invalid challenge, then their bond will be "slashed" i.e. deleted or redistributed to the honest party.

Whether a rollup is a validity rollup or an optimistic rollup, the security of the rollup is strongly dependent on its relationship to its parent chain. This relationship gives a rollup its two key features that other "offchain" protocols lack: inherited double-spend security and a secure two-way bridge.

See Appendix A for a table comparing validity rollups to other offchain protocols.

## Section 2. The validity rollup user experience

### Section 2.1 Deploying and using a validity rollup

To create a validity rollup, a protocol is implemented on the parent blockchain that defines how the validity rollup works. This protocol includes details such as rollup consensus rules, how to verify that rollup blocks follow the consensus rules, how rollup block producers are selected in case there are multiple competing block producers, a protocol for moving assets from the parent chain to the rollup and back to the parent chain again, etc. The rollup protocol is often implemented as a smart contract that is deployed to the parent chain, however the rollup could also be implemented directly in the consensus rules of the parent chain, making it an "enshrined rollup".[50]

Once the rules of the rollup are defined on the parent chain, the genesis block of the rollup can be published and the block producer(s) can begin advancing the state of the rollup. With each rollup block that gets produced, the rollup transaction data is stored in a state update transaction on the parent chain along with a cryptographic proof of the validity of the rollup block (the "validity proof" — which may or may not also be "zero knowledge" in nature). Once the state update transaction is confirmed on the parent chain, the rollup block is considered confirmed by rollup full nodes and is added to the rollup chain tip. In order to re-organize a rollup block that has been confirmed on the parent chain, the parent chain block that the rollup block was confirmed in would have to be re-organized too. This gives rollups their _inherited double-spend security_ feature.

To begin using a rollup, a new user has two options: either transfer an existing asset from the parent chain to the rollup, or receive a transfer from someone who already owns an asset on the rollup. Transferring an asset from the parent chain to the rollup is straightforward: the user sends a deposit transaction to the rollup contract on the parent chain specifying the asset and amount to deposit from their address, along with the rollup address that should receive the deposit. Once the deposit transaction is confirmed on the parent chain, the user will receive their asset in their specified rollup address and can then freely transfer the asset to any other rollup address as they would if they were using the base layer blockchain.

The ability to freely transfer assets on a rollup to any other rollup address makes it easy to onboard new users who either don't have assets on the parent chain or don't want to pay the cost of transferring assets from the parent chain to the rollup. Receiving a transfer from someone who already owns assets on the rollup is as easy as sharing a rollup address and waiting for a rollup block confirmation. There is no need for a new rollup user to first establish inbound liquidity as they would have to if they were using a state channel protocol such as the Lightning Network.

> **Optional feature: Instant confirmations**
> 
> To fully inherit the double-spend security of the parent chain, a rollup block has to be confirmed in a parent chain block. This ties rollup transaction confirmation latency to that of the parent chain. So if, for example, the parent chain block time is on average ten minutes, then rollup blocks will be confirmed no faster than ten minutes on average. 
> 
> To provide a better user experience, rollup block producers can provide ~instant confirmations backed by cryptoeconomic security guarantees. Block producers will first deposit collateral into a smart contract as a security bond. When the block producers receive a transaction to include in a block they will then give the transaction recipient a signed receipt promising that the transaction will be included in the next block. If the transaction is not included in the next block as promised, the block producers who signed the receipt will have their collateral redistributed by the security bond smart contract, with some of the collateral used to compensate users whose transactions weren't confirmed as promised, and any leftover collateral being burned.
> 
> The economic penalty provides transaction recipients a strong guarantee that pending transactions cumulatively worth less than the value of the security bond will be confirmed as promised. Recipients can thus treat the signed receipt from block producers "as good as cash", providing ~instant confirmation for their transaction. [51]

To transfer assets on the rollup back to an address on the parent chain, a rollup user submits a withdrawal transaction to the rollup, specifying the asset and amount to withdraw, along with the parent chain address that should receive the withdrawal. Once the withdrawal transaction is confirmed on the rollup, the withdrawn asset will be deleted from the rollup ledger and the rollup smart contract on the parent chain will send the withdrawn asset to the specified parent chain address.

If the rollup block producers are uncooperative, for example by refusing to include the user's withdrawal transaction in a rollup block, the user can unilaterally withdraw their funds from the rollup. They can use the rollup block data that has been stored on the parent chain to create a validity proof that will convince parent chain full nodes that they currently own a certain amount of the asset they want to withdraw, then push their withdrawal transaction through by submitting it directly to the rollup smart contract on the parent chain. By storing rollup state root updates and rollup block data on the parent chain, and using validity proofs to ensure the validity of withdrawals, validity rollups offer their users a _secure bridge_ to the parent chain (and by extension, other validity rollups built on the parent chain).[52]

A validity rollup bridge gives users two important guarantees: 1) only users who can cryptographically prove ownership of an asset can transfer that asset on the rollup or withdraw the asset back to the parent chain, and 2) as long users can get a transaction confirmed on the parent chain, users can always advance the state of the rollup, including unilaterally exiting the rollup back to the parent chain, even if rollup block production has otherwise completely halted for whatever reason. These guarantees give assets on the rollup the same level of ownership security as assets owned on the parent chain, giving validity rollups the strongest level of ownership security possible.[53]

### Section 2.2 Case study: Passing the #tBTCzkSyncTorch

One of the first multi-user applications of a validity rollup was #tBTCzkSyncTorch, a social media game started by Eric Wall in September 2020. The game was simple: Wall deposited some TBTC ("trustless BTC" on Ethereum) into zkSync, a "Layer 2" validity rollup, then told his followers,

> To receive the torch (worth $100, redeemable for real mainnet BTC protected by ETH collateral), simply post your Ethereum address. No setup required. I will then transfer the torch to someone I trust won't run away with it. You will then do the same. We'll see how far it goes.[54]

**Figure 1. A visualization of the #tBTCzkSyncTorch**

![figure1_vr](https://user-images.githubusercontent.com/9424721/186951525-a98ec22c-7d83-4b0d-8a4f-827fcb794e99.png)

> Image source: [55]

Wall then picked an address from someone he trusted and sent some TBTC to the address using zkSync. The recipient posted a similar message on their own Twitter account, and transferred "the torch" to the next address. And so on and so on, until eventually the torch made its way back to Wall. He held onto the TBTC until he had something else to do with it. 

In February 2022, in the wake of the government of Russia's invasion of Ukraine, the government of Ukraine published an Ethereum address to collect donations from people around the world to assist with their resistance efforts. Wall decided to use the torch funds to make a donation, and submitted a transaction on zkSync to withdraw to the TBTC back to Ethereum Layer 1.

**Figure 2. Eric Wall's zkSync withdrawal as shown in the zkscan.io block explorer**

![figure2_vr](https://user-images.githubusercontent.com/9424721/186952093-51ee6fe8-f8f8-49ee-b4dd-e573e47071a4.png)

> Image source: [56]

At the time, zkSync only had a single block producer operated by zkSync's development company Matter Labs, and Matter Labs was only publishing blocks once every few hours. So about five hours after Wall submitted his withdrawal transaction on the zkSync rollup, the withdrawal was confirmed on Ethereum L1 and he received the TBTC in his L1 address. Wall then swapped the TBTC for ETH and donated the ETH to the Ukrainian government.[57]

Now let's compare the #tBTCzkSyncTorch experience with the social media game that the #tBTCzkSyncTorch was based off of, the #LNTrustChain. The #LNTrustChain game worked similarly: the pseudonymous Hodlonaut started with 100,000 satoshis (or "sats", for short) on the bitcoin Lightning Network, then passed the torch on to someone else, who added a small amount of sats then passed the torch on to someone else, and on and on in a long "trust chain".[58] The trust chain got its name because each recipient of the torch was trusted to not just keep the sats for themselves but to add more sats and pass them along to someone else they trusted to do the same.

Because Lightning Network is a payment channel network, users in the #LNTrustChain had to establish channels between each other or be connected along a path of channels with the correct balances of outbound and inbound liquidity so they could send and receive the torch. There were more than a few cases where this liquidity requirement caused problems for would-be recipients of the torch.[59]

**Figure 3. #LNTrustChain participants seeking inbound liquidity to ensure the torch can be passed.**

![figure3_vr](https://user-images.githubusercontent.com/9424721/186952485-239ab3c3-487e-4b5f-af65-f2f13b4518d4.png)

> Image source: [ibid]

In contrast, #tBTCzkSyncTorch participants never ran into any liquidity issues. By design, such liquidity issues could not have occurred. Rollups function like a blockchain, where users can send payments worth up to the full balance of their wallet with no concern for routing liquidity, and users can receive payments with no inbound capacity limit. There is no need to lock up capital or manage inbound/outbound liquidity to send and receive payments on a rollup, as is required in payment channel networks such as Lightning. These features make validity rollups more capital efficient and, at least for the new user, more user-friendly than Lightning.

It should be noted that this is an apples-to-oranges comparison: zkSync is a rollup that uses a blockchain data structure with global state, and Lightning is a payment channel network with peer-to-peer local state. These protocols are superficially similar in that they are both considered "Layer 2" protocols and both facilitate payments. Beyond that, they have important fundamental differences "under the hood". As such, this comparison of the two is not to say that Lightning doesn't have its own benefits compared to validity rollups, such as extremely low cost, highly secure, high-speed offchain transactions. But the challenges in getting Lightning to a usable state for p2p payments, as demonstrated by the #LNTrustChain user experience, reveal the tradeoffs and rough edges inherent to its design, which rollups avoid due to their own fundamental qualities.

A more thorough look at the Lightning Network and how it compares, contrasts, and is complementary to validity rollups (and vice versa) can be found in Section 4.5.

## Section 3. Enabling new functionality

One of the interesting qualities of validity rollups is that they can enable entirely new functionality to be added to their parent chain without the need for any additional consensus rule changes. This is made possible because rather than be aware of the specific consensus rules of the rollup and know how to execute transactions by following those rules, the parent chain full nodes only need to know how to verify the rollup validity proof. So a Layer 2 validity rollup built on bitcoin Layer 1, for example, could implement an execution environment that supports a more flexible smart contract languages or more advanced privacy technologies, with no other changes needed to bitcoin.[60] 

### 3.1 New smart contract languages

Since the early days of bitcoin, additional scripting capabilities have been envisioned to enable more types of assets and smart contracts than bitcoin supports natively.[61] In the years since then, new capabilities have been implemented either as extensions to Script (bitcoin's native scripting language) or using entirely new smart contract languages. Here are just a few of the new smart contract languages that have been proposed:

- Cairo: [https://www.cairo-lang.org/cairo-welcome-on-board/](https://www.cairo-lang.org/cairo-welcome-on-board/)
- Clarity: [https://clarity-lang.org/](https://clarity-lang.org/)
- Huff: [https://docs.huff.sh/get-started/overview/](https://docs.huff.sh/get-started/overview/)
- Lexon: [http://www.lexon.tech](http://www.lexon.tech)
- Move: [https://github.com/MystenLabs/awesome-move](https://github.com/MystenLabs/awesome-move)
- Michelson: [https://tezos.b9lab.com/michelson](https://tezos.b9lab.com/michelson)
- Noir: [https://noir-lang.github.io/book/index.html](https://noir-lang.github.io/book/index.html)
- Simplicity: [https://blog.blockstream.com/en-simplicity-github/](https://blog.blockstream.com/en-simplicity-github/)
- Sway: [https://fuellabs.github.io/sway/v0.18.1/](https://fuellabs.github.io/sway/v0.18.1/)
- Yul: [https://docs.soliditylang.org/en/latest/yul.html](https://docs.soliditylang.org/en/latest/yul.html)

Today, there are two ways that these programming languages could be used in a bitcoin context:

1. Build support for them into bitcoin as an "embedded consensus" layer (e.g. Omni, Counterparty)[62]
2. Build support for them into an alternative blockchain (e.g. an altcoin or a sidechain)[63]

In both cases, these alternative programming languages would not be able to operate on satoshis natively. Instead, a bridge (also referred to as a "two-way peg" in the literature) would need to be built that would lock satoshis on the bitcoin mainchain, release an equivalent amount of satoshi IOUs inside the new execution environment, and implement some mechanism for converting the IOUs back into satoshis on the mainchain.[64] Today, the types of bridges that can be built to and from arbitrarily complex execution environments are limited by bitcoin consensus rules and generally considered less secure than using the bitcoin mainchain directly.[65] With a validity rollup, bitcoin users could experiment with these alternative execution environments while retaining the full ownership security of the bitcoin mainchain, without imposing any additional computational burden on mainchain full nodes.

### 3.2 New privacy protections

Bitcoin's transparency makes privacy protocols built on-chain inherently fragile and therefore inadequate for preserving privacy in the long-run.[66] Offchain Layer 2 protocols such as Lightning and zkChannels have their privacy own challenges and limitations, largely due to the transparency and limited scripting capabilities of the bitcoin base layer these protocols are built on.[67, 68] As a result, bitcoin users must either follow long, detailed guides about how to gain some semblance of privacy, meticulously following each step and hoping other users do the same so that they can preserve their anonymity set, or else surrender their bitcoin to a centralized or federated offchain system to gain stronger, more easily usable privacy elsewhere.[69, 70, 71]

In the years since Satoshi first contemplated how zk proofs could be used to improve bitcoin privacy, cryptographers have invented new protocols that greatly improve the quality and usability of private cryptocurrency transactions. Validity rollups make it possible to implement these new privacy protocols on bitcoin while inheriting the full security guarantees of the bitcoin mainchain. This would provide bitcoin users with state-of-the-art privacy without having to give up self-custody of their satoshis. Additionally, with a flexible enough proof verification system implemented in the bitcoin consensus rules, bitcoin can be future-proofed so that new advancements in privacy protocols can be adopted without requiring any further consensus-level changes to bitcoin.

An example of a new privacy technology that could be implemented is the zero-knowledge, end-to-end encrypted transaction technique first described in the Zerocash whitepaper and later implemented in Zcash.[72] Zerocash end-to-end encrypted transactions (also referred to as "shielded transactions") provide the best privacy possible today, cryptographically hiding the transaction amount, the sender address, and the recipient address. Shielded transactions also offer the largest theoretical anonymity set, with the upper limit being _the total number of shielded transactions that have been made using the shielded protocol_. 

For example, if there have been 1,000,000 shielded transactions so far, the upper limit of the anonymity set of the next transaction is 1,000,000 since each prior transaction could have each been received by a different user. The next sender could therefore be any one of those up to 1,000,000 prior recipients, providing them an anonymity set of 1,000,000 possible users to hide among. (Of course, each prior transaction that is linked to the same unique recipient will decrease the upper limit of the anonymity set. De-anonymization techniques outside of the blockchain, such as at the network layer, must also be taken into consideration.)

In March 2021, the Aztec rollup launched on Ethereum, becoming the first rollup to implement the Zerocash protocol in production.[74] The developers of Aztec refer to the protocol as a "zk-zk-rollup" due to how zk proofs are used to both provide users with strong privacy and prove the validity of rollup transactions to the Aztec smart contract on Layer 1.[75] By moving shielded transactions into a rollup, Aztec is able to offer users significant cost savings compared to performing the same type of transactions on Layer 1. Bitcoin users could similarly gain both strong privacy and cost savings for private transactions by moving to a zk-zk-rollup rather than transacting on the bitcoin mainchain.

**Figure 4. An Aztec shielded transaction as shown in the aztec.network block explorer**

![figure4_vr](https://user-images.githubusercontent.com/9424721/186952862-66044f20-c2cf-4fb6-9018-d2b8254cd884.png)

> Image source: [76]

A comparison of new onchain private transaction techniques that could be implemented in a validity rollup can be found in Appendix B.

Because of their objectively superior privacy protections, for the purposes of the scalability discussion in Section 4, shielded transactions are used as the example in the case where a new private transaction protocol is implemented in a validity rollup on bitcoin.

## Section 4. Scaling improvements

### 4.1 Increasing throughput with validity rollups
James Prestwich has defined scaling in a blockchain context as "validating more transactions on the same hardware".[77] By this definition, validity rollups can improve scaling by increasing the number of transactions that gain the full security of the parent chain with no (or marginal) additional computational costs for parent chain full nodes. Exactly how much validity rollups improve scaling depends on the implementation details. (Note that the transaction sizes and throughput numbers in this section are approximate and could change depending on implementation details.)

If a validity rollup is designed to work like bitcoin, with a UTXO model and no address re-use, then each rollup transaction would be 113.5 weight units (WU). This is equal to the weight of a 1-input-2-output P2WPKH transaction with the witness removed (because the witness is replaced with a validity proof covering all rollup transactions) and assumes the witness discount is applied to all data in the transaction (because parent chain full nodes do not need to execute these transactions or store them in the UTXO set). 

If a validity rollup uses an account model with address re-use (i.e. one address per user) then it is possible to reduce the weight of each rollup transaction down to 12 WU.[78] This is because public keys can be assigned to account numbers on the rollup, which can then be securely represented in a highly compressed format. Then, rather than including the public key or address in each transaction, the sender and recipient can simply be referred to by their account number. Other rollup transaction details that would have used more bytes in a non-rollup transaction can also be removed or compressed in the state update transaction that gets confirmed on the parent chain.

If a validity rollup uses a UTXO-based shielded transaction model, then there are two components to the transaction data that must be accounted for: the encrypted transaction viewing key (530 WU) and join-split data (129 WU), bringing the total weight of each 1-input-2-output shielded rollup transaction to 659 WU. This makes shielded transactions slightly heavier (in WU) than native segwit 1-input-2-output P2WPKH transactions. Given that they potentially have a _much_ larger anonymity set, and that past an anonymity set of "2" transparent P2WPKH transactions become much heavier than shielded transactions, this larger onchain footprint of shielded transactions may be considered an acceptable cost for such a significant improvement in the quality and usability of private transactions.

Comparing each transaction type in terms of WU:

**Table 1. Comparing different transaction types in terms of weight units**
| Transaction type                      | Non-discounted data weight (4x) | Discounted data weight (1x) | Total (WU)                    |
| ------------------------------------- |:-------------------------------:|:---------------------------:|:-----------------------------:|
| Rollup account model                  | 0                               | 12                          | 12                            |
| Rollup 1-input-2-output UTXO          | 0                               | 113.5                       | 113.5                         |
| Mainchain 1-input-2-output P2WPKH     | 453                             | 108                         | 561                           |
| Rollup shielded 1-input-2-output UTXO | 0                               | 659                         | 659                           |

> Table 1 description: Validity rollup account model is a transaction in a highly compressed format.[ibid] Validity rollup 1 input, 2 output P2WPKH is a Layer 2 P2WPKH transaction with the 27 byte witness removed (because the witness is replaced with a validity proof covering all rollup transactions) and the witness discount applied to all data in the transaction (because parent chain full nodes do not need to execute these transactions or store them in the UTXO set).[79] Mainchain 1 input, 2 output P2WPKH is a Layer 1 native segwit transaction on the bitcoin mainchain.[ibid]

To visually compare each transaction type in terms of "multiples of the smallest possible rollup transaction", we can represent them as blocks as in Figure 5.

**Figure 5. Visually comparing like transactions in terms of multiples of the smallest possible type of rollup transaction.**

![figure5_vr](https://user-images.githubusercontent.com/9424721/186953434-23ea23fe-f662-4847-9fb6-d9b9ac84a44d.png)

> Figure 5 description: Assuming a 12 WU account model validity rollup minimum transaction size, a bitcoin mainchain 1-input-2-output P2WPKH transaction is 46.75x larger than the smallest possible type of rollup transaction that provides equal functionality (sending a payment to a single recipient with some change going back to the sender).

If the parent chain is bitcoin, and the available capacity per block is the same as today (4,000,000 WU or 1,000,000 vbytes) then a bitcoin-like UTXO model validity rollup with no address re-use could enable up to about 3.7 times more transactions to fit per block compared to 1-input-2-output P2WPKH mainchain transactions. This assumes 3,000,000 WU is taken up by the data for all rollup transactions in the rollup block and 1,000,000 WU is left over for the validity proof, transaction script, and miscellaneous other transaction data.[80] Given the same assumptions, an account model validity rollup could enable up to about 35 times more transactions per block. A shielded rollup could fit about 36% fewer transactions per block, but as described in Section 4.1, these transactions would have a much better privacy profile and anonymity set than like-size mainchain P2WPKH transactions, which may make the reduction in transactions per block an acceptable tradeoff.

(Potentially even more UTXO model rollup transactions could fit per block if addresses are made to be even shorter than Layer 1 addresses are today. Francois Grieu has proposed a cryptogram design that is 16 characters in length with up to 117 bits of security.[81] This may be an acceptable security tradeoff if the addresses are storing relatively low amounts of value for relatively short periods of time.)

**Table 2. Comparing how many transactions can fit per bitcoin mainchain block based on a given transaction type**

| Transaction type                      | 1,000,000 WU                          | 3,000,0000 WU            | Total (tx/block) |
| ------------------------------------- |:-------------------------------------:|:------------------------:|:----------------:|
| Rollup account model                  | Validity proof, script, other tx data | 250,000 txs (12 WU/tx)   | 250,000          |
| Rollup 1-input-2-output UTXO          | Validity proof, script, other tx data | 26,432 txs (113.5 WU/tx) | 26,432           |
| Mainchain 1-input-2-output P2WPKH     | 1,782.5 txs (561 WU/tx)               | 5,347.6 txs (561 WU/tx)  | 7,130            |
| Rollup shielded 1-input-2-output UTXO | Validity proof, script, other tx data | 4,552 txs (659 WU/tx)    | 4,552            |

> Table 2 description: Note that these figures assume that the witness discount is applied to all of the transaction data that is part of a rollup block. The mainchain block is broken up into 1,000,000 WU and 3,000,000 WU portions in the table to illustrate how in the case of rollup transactions, some of the weight limit must be used for the rollup validity proof and script data while the rest can be used for the rollup transaction data. 

So far we have only discussed throughput increases in terms of transactions per block based on the currently-available data storage space in each bitcoin block. But unlike bitcoin transactions — even those in a hypothetical world with transaction size optimizations such as cross-input signature aggregation — bitcoin full nodes do not have to execute each validity rollup transaction to determine if they are valid or not. The only computation bitcoin full nodes have to perform to verify a block's worth of validity rollup transactions is to verify a validity proof. A rollup validity proof takes the same amount of time and computational effort to verify whether it is proving the validity of one transaction or a full block's worth of transactions. To take further advantage of this gain in computational efficiency, additional "dumb storage" space per bitcoin block could be provided for rollup transactions. This would enable even more rollup transactions to fit per bitcoin block without any additional CPU burden on bitcoin full nodes.

See Appendix C and D for details about how more throughput and cost savings can be obtained by implementing offchain and offchain-onchain hybrid data availability protocols.

See Appendix E for an investigation into how validity proofs can enable even more throughput increases than validity rollups alone.

### 4.3 Fractal scaling with rollups

The scaling gains provided by validity rollups do not stop at Layer 2 (L2). Validity rollups can be layered on top of validity rollups to provide "fractal scaling" to virtually unlimited levels of scale. Once bitcoin L1 blocks fill up with L2 validity rollup transactions, an L3 validity rollup could be built on top of the L2 validity rollup. Then once the L2 validity rollup blocks fill up with L3 validity rollup transactions, an L4 validity rollup could be built on top of the L3 validity rollup, and so on, and so on.[82]

**Figure 6. Layered rollups**

![figure6_vr](https://user-images.githubusercontent.com/9424721/186954172-43b35f84-0d46-4669-ab66-5aaec0b4a351.png)

> Figure 6 description: An example visualizing how rollups could be layered on top of one another. In this example, there are two Layer 2 rollups: one specialized in providing data availability (perhaps with incentives in place to penalize data unavailability attacks) and another specialized in high-security payments and contracts. On top of the Layer 2 data availability rollup, there are three Layer 3 rollups, each specializing in a different use case: private p2p payments, financial contracts, and in-game asset ownership and transfers. Since the Layer 3 rollups are relying on Layer 2 full nodes for data availability, they could be considered less secure than the Layer 2 rollups that rely on bitcoin Layer 1 full nodes for data availability security. The tradeoff for this reduced security would be lower transaction costs due to the lower cost of data storage on Layer 2. Use cases that require high security could use a Layer 2 rollup such as the high-security payments and contracts rollup here instead.

### 4.4 Scaling validity rollups

As more transactions need to be proven valid at each rollup layer, the difficulty of creating the necessary validity proofs increases. The computation needed to create these proofs can be parallelized using recursive proof composition. Recursion is essentially "proving the validity of proofs", so many computers could work on proving the validity of different transactions and then these proofs could be combined finally into a single proof, allowing horizontal scaling with multiple computers contributing to proving the validity of a rollup block. Trustless recursive proof composition is already possible with both SNARK and STARK proofs.[83, 84] 

**Figure 7. Recursive proof workflow**

![figure7_vr](https://user-images.githubusercontent.com/9424721/186954673-df272a87-73f4-4692-b3e9-b39ec1a95348.png)

> Image source: [ibid]

### 4.5 Scaling the Lightning Network with validity rollups

The Lightning Network is a decentralized network of bidirectional payment channels that routes payments off-chain and uses smart contracts on a base-layer blockchain for dispute resolution and final settlement.[85] Lightning enables any two users connected by a channel route with sufficient liquidity to send and receive value nearly instantly and generally at much lower cost than transacting on the settlement-layer blockchain. To onboard to Lightning in a non-custodial manner, a user must first confirm a transaction opening a channel with another Lightning user. A dual-funded channel can be used to enable bidirectional liquidity in a single transaction.[86] Once a channel is opened, it may later need additional onchain transactions for maintenance such as channel rebalancing or channel closing for final settlement.

The onchain transactions needed to open and settle (and occasionally rebalance) non-custodial Lightning channels result in Lightning usage taking up a measureable amount of limited bitcoin block space. It also results in a hard upper limit on the number of non-custodial users who can be onboarded to Lightning in a given period of time.[87] The additional transaction capacity enabled by validity rollups could be used to support more Lightning transactions, increasing the potential number of users who can onboard and use Lightning in a non-custodial manner. The exact numbers depend on the type of transaction used for the Lightning channel, but for P2WPKH-input-P2WSH-output dual-funded channels, rollups can create room for up to 4.2x more Lightning channel open transactions.

**Table 3. Comparing the size of different types of UTXO model dual-funded Lightning channel open transactions**

| Transaction type          | Non-discounted data weight (4x) | Discounted data weight (1x) | Total (WU) |
| ------------------------- |:-------------------------------:|:---------------------------:|:----------:|
| Rollup LN channel open    | 0                               | 197.5                       | 197.5      |
| Mainchain LN channel open | 795                             | 211                         | 1006       |

> Table 3 description: The rollup transaction is a 2-P2WPKH-input-2-P2WSH-output transaction with the witnesses stripped from each input. The mainchain transaction is a 2-P2WPKH-input-2-P2WSH-output transaction with the witnesses stripped from each input.

**Table 4. Comparing the per-block capacity of different types of UTXO model dual-funded Lightning channel open transactions**

| Transaction type          | 1,000,000 WU                          | 3,000,0000 WU            | Total (tx/block) |
| ------------------------- |:-------------------------------------:|:------------------------:|:----------------:|
| Rollup LN channel open    | Validity proof, script, other tx data | 15,190 txs (197.5 WU/tx) | 16,807           |
| Mainchain LN channel open | 994 txs (1006 WU/tx)                  | 2,982 txs (1006 WU/tx)   | 3,976            |

> Table 4 description: The block is broken up into 1,000,000 WU and 3,000,000 WU portions to illustrate how in the case of rollup transactions, some of the weight limit is reserved for the validity proof and script data while the rest can be used for the rollup transaction data.

The ability to layer validity rollups on top of one another also means that capacity for Lightning can be added as-needed. Each time a rollup reaches its maximum capacity, another rollup could be deployed as an additional layer, then Lightning onboarding can continue until that layer fills up, and so on.

Although Lightning is most famous for settling on the bitcoin mainchain, Lightning channels can be opened on any Lightning-compatible blockchain. Additionally, if there is a Lightning node with channels open on two or more chains, it can route payments between users of these different chains, as if those users had channels open on the same chain. In cases where there are different assets being sent and received through Lightning channel routes (e.g. BTC and LTC) there is an "inadvertent call option problem".[88] However, a Lightning channel route that uses the same asset throughout the entire route, even if the route crosses multiple chains, does not suffer from this problem, or at least the problem is negligible when accounting for any small price differences between the different versions of the same asset on different chains. So users who have Lightning BTC channels open on bitcoin Layer 1 could seamlessly transact with users who have Lightning BTC channels open on Layer 2+ validity rollups provided there is a route with sufficient liquidity between the sender and recipient.

**Figure 8. Crosschain Lightning transactions**

![figure8_vr](https://user-images.githubusercontent.com/9424721/186956813-9e2997ce-94b7-46ba-9a7b-ddc8303b5e5d.png)

> Figure 8 description: There three Lightning nodes: Lightning node "A" that has a channel open on the bitcoin mainchain, Lightning node "B" that has channels open on both the bitcoin mainchain and a Lightning channel rollup, and Lightning node "C" that has a Lightning channel open on the rollup. B has a dual-funded channel open with both A and C, and can therefore route payments between them, even though A and C each have channels open on different layers.

## Section 5. Building validity rollups on bitcoin

As of the time of writing, nearly every validity rollup deployed to production has been built on blockchains that support smart contracts written in a Turing-complete programming language. The flexibility of Turing-complete programming languages opens a wide design space that rollup developers have taken advantage of to program ideosyncratic features and limitations into the rollup smart contracts, such as scripting capabilities, deposit limits, and upgradeability. The flexibility also means that as the validity proof technology improves and new ways to implement or optimize certain features are discovered, rollup developers can upgrade their smart contracts or deploy new ones to keep up with the state of the art.

Despite the popularity of using Turing-complete programming languages to build rollup smart contracts, it would be possible to build a validity rollup on bitcoin using bitcoin's native Turing-incomplete programming language, Script, with relatively small changes (in terms of code footprint) to the opcodes Script supports. In March 2022, Trey Del Bonis published a post describing in detail how a validity rollup on bitcoin could work.[89] According to Del Bonis, the changes needed to support validity rollups on bitcoin are a few extra opcodes enabling the two main primitives of his rollup design — validity proof verification and recursive covenants. And while not strictly required, there are other changes that Del Bonis says would significantly reduce costs and make the rollup more efficient, such as OP_EVAL and PUSHSCRIPT opcodes and increasing or even completely removing the stack element size limit.

Giving bitcoin full nodes the ability to verify a validity proof is an obvious change that is needed to support validity rollups, since validity proofs are an essential part of how validity rollups work. For this component, whoever writes the code to enable validity proof verification in bitcoin will need to make some decisions about the types of rollups they want to enable. Implementing the ability to verify proofs of more complex programs will enable rollups with more capabilities (e.g. more expressive smart contracts, like Rootstock or Stacks) while simpler proofs would enable rollups with fewer capabilities (e.g. simple payments and limited opcodes, like Liquid or bitcoin).

Recursive covenants are a type of smart contract that restricts the type of script that satoshis can be sent to once they are spent. Del Bonis uses recursive covenenants to ensure that satoshis that are locked in a rollup script and haven't been withdrawn by their owner yet remain in the script from one rollup state update to the next. Once the owner of satoshis on the rollup confirms a valid withdrawal transaction on the rollup, then they can exit the recursive covenant script with their satoshis to the Layer 1 withdrawal address they specified.

Recursive covenants are a change to Script that has long been considered by the bitcoin community. As of February 2022, Dave Harding reports that, "I believe the last time the merits of allowing recursive covenants was discussed at length on this list, not a single person replied to say that they were opposed to the idea."[90] However there are currently no specific proposals that have achieved broad consensus among the bitcoin developer community to implement recursive covenants.

The OP_EVAL and PUSHSCRIPT opcodes are nice-to-haves that reduce the size of the rollup script in some areas, reducing the amount of blockspace used and therefore making rollups cheaper to use, all else being equal. Increasing or removing the stack element size limit is another nice-to-have that makes rollups cheaper to use, in this case by increasing the number of transactions that can fit into each rollup state update, thereby enabling the cost of the rollup update to be shared by a larger number of transactions.

The Del Bonis rollup design is one way to build validity rollups on bitcoin, but not the only way. For example, it would be possible to add an extension block to bitcoin with custom logic that supports the creation of specific or arbitrary rollup designs. In his post, Del Bonis discusses several alternative ways that rollups could be built on bitcoin, either as minor tweaks to his more detailed design or using entirely different mechanisms for ensuring the security of funds held in the rollup. Rather than add direct support for the opcodes needed, support for validity rollup primitives could be implemented in Simplicity using Jets, for example.[91]

What is clear is that it is possible to build validity rollups on bitcoin. Some implementations would be more technically difficult to achieve than others, but even with the simpler implementations proposed, bitcoin users stand to gain significant scaling benefits and potentially privacy and other desirable functionality as well.

## Section 6. The costs and risks of validity rollups

While the benefits validity rollups can bring to bitcoin in terms of enabling increased transaction throughput, better transaction privacy, and greater flexibility in the ways satoshis can be encumbered all sound good on paper, these benefits are not without cost or risk. In addition to the usual costs (developer review time, user testing time, etc) and risks (chain split, BTC price decrease, etc) associated with software updates and bitcoin consensus changes in particular, rollups have their own unique costs and risks that need to be considered. This section will examine the most obvious costs and risks uncovered while preparing this report, though others may exist or come up in the future that are not coverd here.

### 6.1 Increased bandwidth and storage costs

If block space is not increased to allow for more rollup transactions, then adding validity rollups to bitcoin will not result in any inherent increase in bandwidth or storage costs for Layer 1 full nodes. The same block space and bandwidth will instead be used more efficiently to pack in more transactions for the same bandwidth and storage costs.

If block space _is_ increased to allow for more rollup transactions, this will increase bandwidth and storage costs for Layer 1 full nodes. More data will need to be relayed around the bitcoin network when broadcasting transactions and blocks. More data will also need to be stored on disk when a block containing rollup transaction data gets added to the blockchain. This is straightforward to measure depending on how much the block space limit is increased to make room for more rollup transactions. See Section 4.1 for data cost calculations per rollup transaction.

### 6.2 Managing full node validation costs

Regardless of whether or not block space is increased to allow for more rollup transactions, validity rollups do impose one new cost on Layer 1 full nodes: the cost to verify the validity proof of the rollup state update. This verification cost can vary widely depending on the complexity of the proof. Benchmark verification times for modern proofs are surprisingly difficult to come across. During my research I found proof verification times ranging from 5ms for a two-year-old PLONK-based SNARK to 2ms for a two-year-old STARK.[92, 93] 

According to benchmarks posted to the Bitcoin Wiki, a bitcoin transaction takes about 0.125ms to verify on a quad-core i7 CPU, which is about 16 times faster than the 2ms verification time of a two-year-old STARK.[94] So if 16 transactions go into a validity rollup state update, then the rollup will break even on verification costs compared to a single Layer 1 bitcoin transaction.

If support for validity rollups is implemented, developers will have to consider an adversarial worst-case scenario where an attacker packs the maximum possible number of validity proofs into a block to try to maximize its verification cost. This would make the block more difficult for weaker full nodes to verify and slow its propagation through the network.

If we consider the base cost for the rollup state update transaction (verification key + proof + script size) to be 2848 WU, as Del Bonis estimated, that equals a maximum 1404 rollup state update transactions per 4,000,000 WU bitcoin block.[89] At a verification time of 2ms per rollup state update, that is a total verification time of 2.8 seconds per block. The block with the longest verification time currently on record is block #364292, which contains a single non-coinbase transaction that takes ~1 second to verify.[95] So the worst-case scenario for verifying a block full of validity rollup updates is about three times slower than the slowest-to-verify bitcoin block currently on record.

If developers were to implement support for validity rollups in bitcoin, they may also want to somehow limit the number of validity proofs that can go into each block, so they can limit the worst-case verification outcome should a block be stuffed full of them. There is certainly a balance to strike here between the number of rollup state updates allowed per block and the additional verification costs imposed on Layer 1 full nodes.

### 6.3 Miner extractable value

Bitcoin was launched under the premise that miners are economically rational, self-interested profit maximizers, and that their profit-maximizing behavior will lead to the emergence of a well-operating peer-to-peer electronic cash system. This design has worked pretty well so far. Rumors of a double-spend attack (which turned out to be a non-event) resulted in swift financial retribution, cautioning miners not to ever deviate from the norm against intentionally re-organizing the blockchain for selfish gain.[96] Even the mere possibility of being able to pull off a so-called "51% attack" has led miners to redistribute their hashpower away from dominant mining pools.[97]

With that said, it is normal and expected that miners operate within the norms of bitcoin to maximimize their profit. Putting only transactions that pay the highest fees into their blocks, to the exclusion of lower-fee-paying transactions, for example, is tolerated and even celebrated.[98, 98] However in the past few years we have seen new miner behavior emerge, particularly in blockchains that support financial contracts such as borrowing and trading. This behavior has come to be known as "non-fee-based miner extractable value" or "MEV" for short.[100] MEV encompasses a growing number of different types of value that miners can extract by ordering transactions in a block to their exclusive benefit (and generally at a cost to other blockchain users).

One example is the "sandwich attack", a form of frontrunning that can be performed against users of onchain automated-market-maker-based algorithmic exchanges. It works like this: A miner observing the mempool sees Alice place a "market buy" order for Asset ABC on the AMMSwap exchange. The miner will place their own equal-sized market buy order for Asset ABC on AMMSwap, yielding the miner an average cost basis of X. The miner structures their block so that their market buy order is immediately before Alice's market buy order in the block. As a result, when Alice's market buy order executes she receives Asset ABC at a cost basis of X+1. In the same block, the miner then places an equal-sized AMMSwap market sell order for Asset ABC right after Alice's market buy order in the block, capturing the "+1" liquidity that Alice's buy order gave to Asset ABC. The end result: Alice paid more for Asset ABC than she otherwise would have, and the miner earned +1 profit risk-free.

MEV is being applied across all kinds of different scenarios, including arbitrage, liquidations, slashing penalties, token sales, and more. Given all of the MEV that is happening on blockchains that support advanced financial contracts, and given that validity rollups have the ability to enable such contracts to be used in Layer 2 rollups build on bitcoin, perhaps the most pertinent questions for the bitcoin community to consider before implementing support for validity rollups on bicoin are: do validity rollups on bitcoin create opportunities for MEV where they otherwise do not exist? And if so, would the MEV opportunities created weaken the security of bitcoin Layer 1?

Although it is possible to build financial smart contracts on bitcoin using embedded consensus layers such as CounterParty and Omni (or even native BTC using DLCs) this type of usage hasn't taken off to the same degree as it has on other blockchains such as Ethereum, BSC, Solana, etc. If bitcoin enables support for validity rollups, it's possible that whatever shortcomings have held back the development and adoption of financial smart contracts on bitcoin could be addressed, increasing the likelihood that MEV will occur on bitcoin.

To answer the question of whether validity rollups on bitcoin create new opportunities for MEV, we must first be specific about what kind of validity rollups we are enabling on bitcoin. It is possible to limit the expressivity of the scripting capabilities supported by validity rollups that can be built on bitcoin by limiting the complexity of the proofs that bitcoin full nodes are able to verify. If the bitcoin community wanted to, they could limit validity rollups to being no more (or not much more) expressive than bitcoin is today. This would most likely not lead to any new MEV opportunities being introduced.

The bitcoin community could also decide to enable more expressive validity rollups to be built on bitcoin. Perhaps these rollups would be expressive enough to enable the types of contracts that are vulnerable to MEV. In this case, there would be new MEV opportunities created on bitcoin. This MEV would mainly be captured by rollup block producers on Layer 2. Due to bitcoin's long block time, it would be relatively risky for Layer 1 miners to try to re-org blocks in order to capture some MEV from Layer 2 due to the high cost of mining a block. Even on blockchains such as Ethereum that have relatively short block times, there have been no reports of miners re-orging Layer 1 blocks to capture MEV on Layer 2. TBD how or if this changes as Layer 2 rollups transition to decentralized block production.

Several developers and researchers were asked about this while doing interviews for this report and the consensus is that MEV on bitcoin validity rollups may lead to an increase in Layer 1 transaction fees due to the increased transactional volume created by MEV bots, but otherwise Layer 1 users would not be affected by MEV. Those familiar with the matter pointed to the lack of negative effects of Layer 2 rollups on Layer 1 Ethereum users as evidence for why bitcoin Layer 1 users would likely not be negatively effected by Layer 2 rollups built on bitcoin either. Given that Layer 2 rollups are a relatively recent phenomenon on Ethereum, however, more time and research is needed to better understand the interplay between MEV on Layer 2 and consensus security and incentives on Layer 1.

Researchers have been able to develop many solutions that prevent or minimize the negative effects of MEV. Some of these solutions are structural changes to consensus that impede the ability of block producers to order transactions in their favor.[101] Other techniques hide transaction information so that block producers and "searchers" are unable to see transactions that would be vulnerable to MEV.[102] Users who don't want to have their lunch eaten by MEV bots can and should demand that developers implement these MEV countermeasures in their rollups and rollup-based financial applications. If these countermeasures become widely implemented then MEV as a general concern could become a thing of the past.

### 6.4 Harmful smart contracts

Enabling validity rollups could have unintended negative side effects aside from MEV in the form of "harmful smart contracts". These are smart contracts that could be used to incentivize miners to attack each other or specific users, disrupting the normal incentives of Nakamoto consensus. This section will look at a few examples of harmful (or potentially harmful) smart contracts that could be enabled because of or alongside validity rollups on bitcoin. This is not an exhaustive catalogue of such contracts. The most important takeaway is that there are risks both known and unknown (with the possibility of unknown risks being a risk itself) associated with enabling more advanced scripts on decentralized blockchains such as bitcoin.

**TxWithold contracts**
One example of a harmful smart contract is a TxWithold contract. In a BitMEX Research article, Gleb Naumenko describes how covenants can be used to build a smart contract that incentivizes miners to withold (i.e. not confirm) a transaction for a certain number of blocks.[103] In the Del Bonis rollup model described in Section 5, recursive covenants are a prerequisite for enabling validity rollups on bitcoin.[89] Even if the rollups enabled are relatively limited in capability e.g. no more capable of expressive contracts than bitcoin is today, by enabling recursive covenants on bitcoin Layer 1, certain TxWithold smart contracts could be possible. Exactly how much harm these kinds of contracts could do in practice is an area where further research is needed.

**Re-org wars**
If a validity rollup on bitcoin supported expressive smart contracts, they could be used to instigate "re-org wars", where smart contracts battle each other to incentivize and dis-incentivize miners reorganizing the blockchain.[104, 105] A similar attack was shown to be possible using a "HistoryRevision contract" on Ethereum Layer 1.[106] It's unclear if such contracts deployed to Layer 2 could have the same effect on Layer 1, or whether it _would_ be possible but only under _certain conditions_ (e.g. heavy overlap of block producers on both layers). Even if such reorg contracts on Layer 2 could have the same effect on Layer 1, if they can both incentivize and dis-incentivize reorgs then perhaps they cancel each other out and there's no harm done. Again, this is an area where further research is needed.

**SPV bridges and optimistic contracts**
One category of smart contract whose positive uses have been discussed at length by researchers but whose potential for harm has arguably been under-explored is the "SPV bridge" (also referred to as a "hashrate escrow") and the similar category of "optimistic" smart contracts.[107, 108] These smart contracts are similar in that their users "trust" the hashpower majority to not "steal" funds held in the smart contract. In this context, it is not the potential for theft by miners that makes the smart contract harmful (though that would of course be harmful to the specific victims of the theft). Rather, it is _the enabling incentive to collude to effectuate the theft_ that makes the contract potentially harmful _to the security of the blockchain itself_. By creating an incentive to collude to form a hostile majority where no such incentive would otherwise exist, the SPV bridge and similar contracts can be considered to be directly undermining the otherwise honest, correct, incentive-compatible operation of Nakamoto consensus.

Implicit in the categorization of such SPV bridge and optimistic contracts as harmful is the assumption that they do in fact incentivize miners to collude to form a hostile majority. However empirical observation has shown that miners have not yet colluded to steal despite such contracts existing, suggesting there is some other counter- or dis-incentive preventing them from doing so.[109] It's also possible that there's simply a tipping point that has yet to be reached and it's only a matter of time before the collusion and theft these contracts incentivize finally occurs. With such contracts having already been deployed in the wild, only time will tell what harm or lack thereof they do to their host networks. Once again, more time and research is needed.

### 6.5 Provoking authoritarians
While the very existence of bitcoin itself is a provocation toward authoritarians who seek to control how other people earn, save, and spend money, validity rollups could add whole new provocative dimensions to bitcoin.

**Private payments**
Shielded transactions would enable peer-to-peer transactions that are nearly as private and untraceable as physical cash. In some cases, shielded transactions are potentially even more private and untraceable than physical cash, due to the possibility for remote, anonymous interactions online. This would give people under an authoritarian regime (in their government or in their home) the ability to earn and spend money privately, using a hidden wallet on a small computing device that either would not raise suspicion or could be concealed more easily than physical cash.

**Financing a freer future**
Censorship-resistant smart contracts provide permissionless access to finance outside of authoritarian control, enabling entrepreneurs to raise capital for their businesses, activists to fund their movements, and oppressed minorities to circumvent the financial controls forced on them by the authoritarians in their lives. Control of finance is control of a huge part of the economy and people's livelihoods. Providing a neutral, programmable, globally accessible infrastructure for financial activity can give people under the thumb of authoritarians a way to build wealth and fund a freer future for themselves and their communities.

**Really free speech**
Decentralized domain name systems provide websites with censorship-resistant memorable identities. When combined with shielded bitcoin used to pay file hosts on decentralized filesharing protocols such as Bittorrent or IPFS, it becomes possible to create websites that are nearly impossible to take down. In societies where activists and journalists are targeted for the information they publish, this could be a real thorn in the side of authoritarians.

**To be (a powerful and private smart contract platform) or not to be...**
Giving people new tools to fight authoritarians is a good thing. The risk is that these powerful new tools invite more negative attention to bitcoin. Where freedom vs authoritarianism is concerned, the anti-authoritarian bitcoin community may be willing to say, "bring it on". But these powerful new tools enabled by validity rollups can be used for harmful purposes as well. Just as private payments can be used by someone to escape the prying eyes of their abusive domestic partner, private payments can also be used to facilitate untraceable ransom payments. Just as a censorship-resistant website can be used to host an uncomfortable political truth, it can also be used to host nonconsensual material that victimizes innocent people. The bitcoin community already has to deal with these kinds of concerns, but validity rollups could amplify them even further.[110, 111]

This is another one of those issues where if validity rollups are enabled, the bitcoin community will have to make a decision about exactly _what kind_ of rollups will be enabled: does the bitcoin community invite the full contents of Pandora's box to be released on Layer 2, or are there certain features or smart contracts that bitcoin users should never be allowed to access in a trustless manner? I phrase the question that way, with an emphasis on "trustless" because the alternative to supporting these powerful and controversial use-cases directly on bitcoin (via a Layer 1 extension or as a new protocol on Layer 2) is not that these use cases will never happen and all risk and harm from such use cases will be averted. The alternative is the status quo, where bitcoin users must give up control of their satoshis to trusted "bridges" in exchange for IOUs on sidechains or altcoin chains where these use cases are actually supported.[64] On this question, the choice before the bitcoin community isn't between _risk_ or _no risk_, it's between _risk without trusted third parties / with fewer trusted third parties_ or _risk with trusted third parties / more trusted third parties_.

### 6.6 Novel cryptography

The novelty or Lindy factor of the cryptography used in validity proofs depends on what kind of validity proof is being referred to. STARK-based validity proofs rely on the oldest cryptographic primitives and have the weakest security assumptions of the various validity proof systems. SNARK-based validity proofs rely on newer cryptographic primitives such as elliptic curves (as in bitcoin) and have stronger security assumptions.[112]

**Figure 9. The Cryptographic Proof Family Trees - Age and cryptographic assumptions**

![crypto-family-tree](https://user-images.githubusercontent.com/9424721/186959627-1f09a5d3-a4d5-4d06-980b-bd0791bd1543.png)

> Image source: [ibid]

Another consideration is the age of implementation and "battle hardening" that a given validity proof implementation has gone through. With the oldest production implementations now several years old (about as old as the Schnorr algorithm implementation that was integrated into Bitcoin Core) there are several validity proof systems that could be used or at least looked at for inspiration.[113] Though it should still be noted that the validity proof designs the implementations are based on are relatively new compared to the Schnorr signature algorithm.

---

### Appendix A. Comparing validity rollups to other protocols

Validity rollups are part of a decade-long history of protocols designed to improve the scalability and capabilities of blockchain-based digital asset and smart contracts.

We can see in Table 5 how validity rollups are designed with a unique combination of onchain data availability and validity proofs, giving them no channel or denomination limit and ownership security equal to that of their parent chain.

**Table 5. Protocol comparison table**

| Protocol                  | Subcategory                  | Bridge security model     | Data availability                         | Channel/denomination limit | Unilateral exit | Parent chain miners can steal* |
| ------------------------- |:----------------------------:|:-------------------------:|:-----------------------------------------:|:--------------------------:|:---------------:|:------------------------------:|
| CoinWitness               |                              | Custodial or federated    | Offchain (self-custodial)                 | No                         | Yes             | Yes                            |
| State channel [a],[b]     |                              | Fault proof               | Offchain (self-custodial)                 | Yes (Channel)              | Yes             | Yes                            | 
| Statechain [c]            |                              | Semi-custodial            | Offchain                                  | Yes (Denomination)         | Yes             | No                             |
| Plasma [d]                |                              | Fault proof               | Offchain (self-custodial)                 | No                         | Maybe**         | Yes                            |
| Rollup                    |                              |                           |                                           |                            |                 |                                |
|                           | Validity rollup  [e]         | Validity proof            | Onchain                                   | No                         | Yes             | No                             |
|                           | Optimistic rollup [f]        | Fault proof               | Onchain                                   | No                         | Yes             | Yes                            |
| Validia chain***          |                              |                           |                                           |                            |                 |                                |
|                           | Adamantium [h]               | Validity proof            | Offchain (primary), onchain (fallback)    | No                         | Yes             | No                             |
|                           | Validium [g]                 | Validity proof            | Offchain                                  | No                         | Maybe           | No                             |
|                           | Validity valutia [i]         | Validity proof            | Offchain (Collateralized data custodians) | No                         | Maybe           | No                             | 
|                           | Volition [g]                 | Validity proof            | On or offchain (user-configured)          | No                         | Maybe           | No                             |
| Optimistic valutia [j]    |                              | Fault proof               | Offchain (Collateralized data custodians) | No                         | Maybe           | Yes                            |
| Softchain [k]             |                              | Proof of Work fault proof | Offchain                                  | No                         | Yes             | Yes                            |
| Sidechain [l]             |                              |                           |                                           |                            |                 |                                |
|                           | Optimistic sidechain [m]     | Fault proof               | Offchain                                  | No                         | Yes             | Yes                            |
|                           | Drivechain [n]               | Hashrate escrow           | Offchain                                  | No                         | Yes             | Yes                            |
|                           | Collateralized sidechain [o] | Collateralized multisig   | Offchain                                  | No                         | No              | No                             |
|                           | Federated sidechain [p]      | Federated multisig        | Offchain                                  | No                         | No              | No                             |
|                           | Centralized sidechain [q]    | Custodial                 | Offchain                                  | No                         | No              | No                             |
|                           | Zendoo[r]                    | Validity proof            | Offchain with onchain checkpoints         | No                         | Maybe**         | No                             |

> Table 5 notes:
> [ * ] "Parent chain miners can steal" here refers to the idea that if enough parent chain block producers aka "miners" collude then they can steal money from users of the protocol in question without either the cooperation of the user being stolen from (as would be necessary if they were censoring and extoring the user) or needing to re-org a parent chain block (as would be necessary if they were double-spend attacking the user).
> 
> [ ** ] "Maybe" in this column refers to the fact that unilateral exit is possible only if the necessary data is available to users who need it to create their withdrawal transaction.
> 
> [ *** ] See Appendix C for more details about validia chains.

> Table 5 references:
> [a] https://lightning.network/lightning-network-paper.pdf
> 
> [b] https://l4.ventures/papers/statechannels.pdf
> 
> [c] https://medium.com/@RubenSomsen/statechains-non-custodial-off-chain-bitcoin-transfer-1ae4845a4a39
>
> [d] https://plasma.io/plasma.pdf Note: There are [many varieties](https://medium.com/onther-tech/plasma-world-map-ba8810276bf2) of Plasma, but they all have the fundamental qualities described in the above table in common.
>
> [e] https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477
>
> [f] https://ethresear.ch/t/minimal-viable-merged-consensus/5617
>
> [g] https://medium.com/starkware/volition-and-the-emerging-data-availability-spectrum-87e8bfa09bb
>
> [h] https://ethresear.ch/t/adamantium-power-users/9600
>
> [i] https://blog.matter-labs.io/zkporter-a-breakthrough-in-l2-scaling-ed5e48842fbf
>
> [j] https://blog.celestia.org/celestiums/ 
>
> [k] https://gist.github.com/RubenSomsen/7ecf7f13dc2496aa7eed8815a02f13d1
>
> [l] https://blockstream.com/sidechains.pdf
>
> [m] https://near.org/blog/eth-near-rainbow-bridge/
>
> [n] https://www.truthcoin.info/blog/drivechain/
>
> [o] https://github.com/nomic-io/bitcoin-peg/blob/master/bitcoinPeg.md
>
> [p] https://www.rsk.co/Whitepapers/RSK-White-Paper-Updated.pdf
>
> [q] https://wbtc.network/assets/wrapped-tokens-whitepaper.pdf 
>
> [r] https://arxiv.org/abs/2002.01847

## Appendix B. Comparing three alternative cryptocurrency privacy techniques

Due to many of bitcoin's current limitations (e.g. limited scripting capabilities) and qualities (e.g. transparent transaction amounts) the efforts to reduce the traceability and improve the privacy of bitcoin transactions are always an uphill battle. Examples of such protocols built for bitcoin include Cahoots, CoinJoin, CoinSwap, PayJoin, TumbleBit, stealth addresses, and variations or spinoffs thereof.

Perhaps the most important weakness of bitcoin that makes privacy difficult is also seen as one of bitcoin's biggest strengths: the transparent nature of transaction amounts i.e. bitcoin's public auditability. Because bitcoin transaction amounts are transparent and publicly visible on the blockchain, they can be used to correlate senders and recipients and distinguish recipient addresses from change addresses and self-sends.[114, 115] Bitcoin privacy protocols try to fight these heuristics by mixing coins with one or more other users and by creating equal-amount outputs to increase the anonymity set of each transaction. But if any participant in the mix does something to de-anonymize themselves, they are removed from the anonymity set (sometimes without other mix participants even knowing). Eventually, an anonymity set can be whittled down to being no larger than a single-output transaction. This gradual degradation of mix anonymity sets makes bitcoin privacy inherently fragile, requiring constant costs (incurred as mining fees) to maintain anonymity set sizes via re-mixing.

The cost of maintaining anonymity set sizes, plus the requirement to wait for a sufficient number of users to mix equal-amount outputs to gain a sufficiently large anonymity set, can make bitcoin privacy techniques impractical for point of sale transactions or transactions of low value. There is justification for hoping that offchain protocols such as Lightning can offer improvemnts: because Lightning transactions are offchain, the details of transactions are only known to each node along a payment channel path, and in some cases only part of the details of the transaction are known to each node along the path. Atomic multipath routing could add even more ambiguity into Lightning payments, since routing nodes would not know whether they are routing the full payment or only part of the payment. However the privacy enabled here is probabilistic at best, and difficult to quantify for both senders and recipients: it could be the case that their counterparties along the path know either very little or everything about their payments. That's not to mention the privacy issues associated with balance probing and channel open/close transactions.[116] In its current form, Lightning is best considered to be a fast and high-throughput payments solution, not a privacy solution.

Researchers have studied the shortcomings of bitcoin privacy protocols and invented new protocols that they believe offer significant improvements to protect user privacy. We will examine three of the most well-known alternative privacy protocols and compare the amount of data that each reveals about its users, thereby providing a basis for comparing their relative privacy improvements.

### Grin 

Grin is the second independent implementation of the Mimblewimble protocol. While Mimblewimble does have some features that could be considered privacy improvements, such as confidential transactions that hide transaction amounts, and non-interactive transaction cut-through which can remove some transactions from the public transaction graph, in their most simple form Mimblewimble transactions still reveal all the information needed to build a public transaction graph. By watching transactions as they are sent over the peer-to-peer network, it is possible for an observer to follow inputs and outputs to build a graph of where coins are coming from and where they are going to. For these reasons, Mimblewimble is most effective as a scaling solution, not a privacy solution. 

**Figure 10. A Grin transaction shown in its most simple form**

![figure10_vr](https://user-images.githubusercontent.com/9424721/186960781-6f0abb70-3efa-4670-863d-9cf1e79875cb.png)

> Image source: [117]

To quote from the Grin documentation:

> ...[I]t is possible to monitor peer-to-peer network activity and obtain the transactions before they're included in a block and aggregated with others. By setting up sniffing nodes connected to many peers, you can figure out which outputs are being spent by what transaction, allowing you to build a partial transaction graph by separating the aggregation done at the block level... As of today, an almost complete transaction graph can be constructed.[ibid] 

Work is ongoing to improve the privacy of Mimblewimble-based cryptocurrencies by combining the protocol with other privacy techniques. See for example Chaidos and Gelfer's Lelantus-MW protocol.[118]

### Monero

Monero launched as a fork of Bytecoin, which was the first implementation of the CryptoNote protocol. The defining privacy features of the CryptoNote protocol are its one-time ring signatures and stealth addresses.[119] Ring signatures enable senders to use outputs from other transactions as "decoy inputs" to obscure the true input or source of their transaction, in effect doing a non-interactive coinjoin with other users of the protocol. Stealth addresses enable recipients to share a static public address that senders then use to derive single-use private addresses, this way the addresses that appear as recipients on the blockchain cannot be linked to the public stealth address originally shared by the recipient.

Monero has since improved on the original CryptoNote protocol by implementing RingCT, a protocol that combines ring signatures and confidential transactions to hide transaction amounts.[120] The addition of RingCT means that inputs no longer need to select equal-amount decoys, which makes it so that the anonymity set of each ring signature can be as large as the user is willing to pay for them to be (or as large as the protocol will allow them to be; as of Monero 0.13.0 all rings must have a size of 11, to ensure uniformity).[121]

Despite Monero's significant privacy improvements compared to bitcoin, the web of transactions created by RingCT and stealth addresses are not as impenetrable as the mass of metadata on the blockchain would first appear. There are attacks based on common usage patterns such as the overseer attack, the flashlight attack, and the tainted dust attack that can be used to deanonymize either the sender or recipient of a transaction.[66] As with bitcoin-based privacy protocols, decoy-based protocols such as that implemented in Monero have been shown to be fragile in practice (though arguably more forgiving to "mistakes" than bitcoin, thanks to the multiple layers of cryptographic hiding and obfuscation).

**Figure 11. How a Monero transaction looks in the explorermonero.com explorer**

![figure11_vr](https://user-images.githubusercontent.com/9424721/186961096-4d5c4718-36c6-4fc3-906c-2bc6058bfbd7.png)

> Image source: [122]

### Zcash

Zcash is the first implementation of the Zerocash protocol. The defining privacy feature of the Zerocash protocol is its use of zk-SNARKs to encrypt the amount, sender, and recipient of specially-crafted "shielded" transactions, fully hiding these details from the public. Rather than refer to specific public accounts or transparent unspent transaction outputs in the blockchain, spenders instead create a zero-knowledge proof that convinces network full nodes that the sender owns and can therefore spend a certain number of "notes" (the encrypted equivalent of unspent transaction outputs) and that the amounts of the encrypted inputs are equal to the sum of the encrypted outputs plus the transparent mining fee.[123]

**Figure 12. How a Zcash shielded transaction looks in the zcha.in explorer**

![figure12_vr](https://user-images.githubusercontent.com/9424721/186961677-5b858ee8-9d44-4208-8d9a-f456693c9581.png)

> Image source: [73]

While Zerocash transactions hide the most transaction information of all currently known techniques, there is still some onchain metadata leakage that could be used to de-anonymize transactions. For example, a Zerocash transaction publicly reveals the amount of inputs and outputs used in the transaction. This has been shown in practice to be enough information to de-anonymize the sender of a Zerocash transaction.[124] While this demonstration was contrived, and there is a way to mitigate this attack by merging the notes together before sending them to their final destination, and the risk of deanonymization decreases the more transactions with like-input-and-output note amounts there are to hide among, the mock attack showed how the little metadata that Zerocash does reveal can be used to deanonymize users. Countermeasures against the onchain (and offchain) metadata leaks that there are must therefore be taken by users (or the wallets of users) who need to remain unidentifiable when transacting with shielded coins.

## Appendix C. Increasing throughput with offchain data availability

Validity rollups are designed to increase transaction throughput and potentially provide alternative execution environments without giving up security relative to holding funds and transacting directly on the rollup's parent chain. As explained in Section 1, this high degree of security is obtained by combining the use of validity proofs with the storage of rollup transaction data inside of parent chain blocks. The validity proofs provide the same ownership guarantees as any other valid digital signature accepted by the parent chain, and the onchain data storage provides the same data availability guarantees that every other transaction stored in the parent chain has.

Storing a rollup's transaction data directly in the blocks of its parent chain has a high cost. If the parent chain is the bitcoin mainchain, that cost is measured most directly in the sats-per-vbyte market fee rate charged to every transaction that takes up mainchain block space. Even if the fee rate is discounted for the space taken up by rollup transaction data — an implementation detail that is possibly justifiable because these transactions are only stored, not executed, by mainchain full nodes — if the market fee rate for mainchain block space is relatively high, then the per-transaction cost to use the rollup can be relatively high as well. Additionally, the block size limit inherently limits the transaction throughput of validity rollups built on bitcoin.

This is where the concept of a _validia chain_ comes in. A validia chain is a blockchain that, like a validity rollup, inherits double-spend security from a parent chain by advancing its state using validity proofs that are sequentially verified by parent chain full nodes. Unlike a validity rollup, rather than exclusively use the parent chain for data availability, validia chains use one or more offchain data availability solutions instead. The tradeoff here is that throughput can be increased beyond bitcoin's block size limit — potentially lowering per-transaction costs as well due to the use of a cheaper data availability solution — at the expense of a possible reduction in the certainty and security of data availability guarantees. Bitcoin could add support for these validia protocols at the same time as support for validity rollups is added by implementing flexible enough to support various offchain data availability solutions, giving users the choice between highly secure but expensive onchain data availability or less secure but less expensive offchain data availability.

The validia chain varieties invented so far are:
- Validium: Offchain uncollateralized data availability
- Validity valutia: Offchain collateralized data availability
- Volition: Per-transaction choice between offchain or onchain data availability
- Adamantium: Choice between custodial or self-custodial offchain data availability, with onchain fallback

("Validia chain" and "valutia" (pronounced "vuh-loo-sha") are new nomenclature developed for this report to classify previously unnamed categories of blockchains. Since the other categories mentioned here had cool names it seemed appropriate to assign a couple of new cool names where they seemed needed.)

Since data availability is no longer provided by the parent chain, validia chains are not considered rollups. However, validia chains do still offer many benefits when compared to alternatives:
- No channel limitations (unlike state channels)
- No denomination limits (unlike statechains)
- Double-spend security is inherited from the parent chain (unlike sidechains)
- Third parties can freeze user assets, but not arbitrarily steal them at no cost (unlike plasma chains, statechains, or uncollateralized sidechains)

(See Table 5 for a more detailed comparison.)

To expand on each of the options:

### Validium

A validium is a validia chain where data availability is provided by one or more uncollateralized third party data custodians. In order for the state of a validium to advance, the data custodian (or m-of-n custodians, if a federation of data custodians is used) must sign the new state root, attesting that they have a copy of the data. If the validium ceases producing blocks for some reason, users can query the data custodian(s) to receive a copy of the data they need to produce a validity proof and unilaterally exit the validium and withdraw their assets back to the parent chain. So long as at least one data custodian has the necessary data, unilateral exit remains possible. If no full copy of the validium state data is available, then the assets of all users of the validium are frozen until the data becomes available again.

The first mainnet Validium deployed was DeversiFi, a noncustodial exchange built on Ethereum.[125]

### Validity valutia

A validity valutia is a validia chain that relies on _collateralized value at stake_ to incentivize data custodians to preserve data availability. Users will lock some asset as collateral in a smart contract in exchange for the right to take on the role of data custodians. The parent chain then verifies data availability attestations from the data custodians with each new valutia state update as it would with a validium.

If at any point data that should be available becomes unavailable, then the collateral of the data custodians is frozen until the data is made available again. The threat of frozen collateral is designed to keep the data custodians honest and give them an incentive to ensure that the data valuum users need to unilaterally exit remains available if needed.

Since data availability is provided offchain, it is expected that storing valutia data will be less expensive than storing it directly on the parent chain, as it would be with a rollup. At the same time, the collateralized value at stake should provide stronger data availability guarantees than a validium's uncollateralized data custodians. This offers users one more option in the tradeoff space of cost and security.

In an April 2021 blog post, Matter Labs proposed the first validity valutia, zkPorter.[126] The design Matter Labs proposed used Ethereum as the settlement layer and a network of zkSync token stakers for data availability. In a February 2022 blog post, Celestia Labs proposed a similar valutia design called a Celestium, which could work with validity proof- or fault proof-secured bridges.[127] Celestia Labs proposed that Celestiums would use Ethereum as the settlement layer and a separate blockchain called Celestia specifically for data availability.

### Volition

A volition gives users a choice on a per-transaction basis between storing data onchain ("rollup mode") or storing data offchain ("validium/valutia mode"). If the user chooses rollup mode, then the data that they need to unilaterally exit the volition is stored inside a parent chain block as it is with a validity rollup. If the user chooses validium/valutia mode, then the data is stored using whatever offchain data availability solution the volition is configured to use.

The per-transaction optionality provided by volitions means that users can tailor the security and cost of their transaction to the given application. For low-stakes applications, such as recreational gaming or low-value payments, validium/valutia mode could be used. For high-stakes applications, such as rare art collecting or high-value trades, rollup mode could be used. To reduce the decision cost, applications could suggest a mode to use depending on the value or context of a transaction, or even make that decision for the user automatically based on a user-configurable setting.

Volitions were first described by StarkWare in their June 2020 post "Volition and the Emerging Data Availability Spectrum".[125] zkPorter was the first proposed implementation of volition.[126]

### Adamantium

An adamantium is a validia where users can choose between validium/valutia mode, self-custodial data availability, or designated third-party data availability, with a fallback protective withdrawal to the parent chain. By default, Adamantiums function as a validium/valutia. However, if the user does not want to rely on the validium/valutia's default offchain data availability solution, the user can opt to become a Power User.

If the user opts to be a Power User, they must sign each new state root the same as every data custodian defined by the validium/valutia, thereby attesting to the fact that the user has a copy of the new state, which is needed to be able to unilaterally exit from the adamantium. Optionally, the user can designate one or more third parties to be a Power User on their behalf, in which case at least one of the Power Users that the user has designated must preserve data availability and sign the new state root on the user's behalf.

If a Power User does not sign the new state root in a timely manner, then their funds and/or the funds of any users they failed to sign on behalf of will be automatically withdrawn to the parent chain. This protects the affected user(s) from unavailability of the new state, ensuring they have access to their funds.

The adamantium model is not risk-free: although Power Users must attest to having the necessary data to enable unilateral exit, it is possible that they don't actually have the data at the time of the attestation, or that they later lose access to the data for some reason. Users must weigh the cost of computing the state and securing backups of this data (or paying someone else to do this for them, and taking on the risk of this outsourcing) against the cost of using a validity rollup that has strong data availability guarantees built-in.

The Adamantium model was first described by Starkware in their June 2020 post "Volition and the Emerging Data Availability Spectrum" under the name "Trustless Off-Chain Data Availability" or "TODA" for short.[125] It was later given the name "Adamantium" and described in-depth by Avihu Levy in the Ethereum research forum in May 2021.[128]

### The data availability security spectrum

**Table 6. Categorizing validity bridges**

| Bridge category           | Data availability model                                                                                    | 
| ------------------------- |:----------------------------------------------------------------------------------------------------------:| 
| Validity rollup           | Data onchain                                                                                               |
| Adamantium                | Data offchain (collateralized, uncollateralized, or self-custody) with fallback withdrawal to parent chain |
| Collateralized Volition   | Data onchain or offchain (collateralized)                                                                  |
| Validity valutia          | Data offchain (collateralized)                                                                             |
| Uncollateralized Volition | Data onchain or offchain (uncollateralized)                                                                |
| Valdium                   | Data offchain (uncollateralized)                                                                           |

> Table 6 description: Categorizing validity bridges based on the data availability solution used. Ordered from most to least security, in the author's humble opinion. Inspired by the table by Brand et al in [125]. Onchain settlement + onchain data availability is a validity rollup, which offers the strongest security guarantees. Onchain settlement + offchain data availability opens a diverse spectrum of options for data availability with varying degrees of cost and security for end users to consider.

## Appendix D. A closer look at validity sidechains

In the October 2014 paper "Enabling Blockchain Innovations with Pegged Sidechains" by Back et al, the authors defined a pegged sidechain as:

> a sidechain whose assets can be imported from and returned to other chains; that is, a sidechain that supports two-way pegged assets. (p. 8)

Back et al also gave pegged sidechains these ownership security requirements:

> 1.  Assets which are moved between sidechains should be able to be moved back by whomever their current holder is, and nobody else (including previous holders).
> 2.  Assets should be moved without counterparty risk; that is, there should be no ability for a dishonest party to prevent the transfer occurring. (p. 5)

The authors go on to specify the design of a sidechain that uses SPV proofs to facilitate the transfer of coins between the parent chain and a pegged sidechain.[53] The problem with the SPV proof design is that it does not actually satisfy the two ownership security requirements defined for pegged sidechains. Requirement (1) is violated because a majority of parent chain hashpower managers (miners or pool operators) would be able to forge SPV proofs to move coins they are not the "current holder" of, according to the current sidechain state, out of the sidechain script and into one or more parent chain addresses of their choosing. This is the "parent chain miners can steal" problem mentioned in Table 5 and also described in Section 6.4 of this report and Section 4.2 of the pegged sidechains whitepaper.[ibid] Requirement (2) is violated because a "dishonest" hashpower majority can censor legitimate withdrawal transactions to prevent users from moving their coins from the sidechain back to a parent chain address.

Validity rollups satisfy requirement (1), but do not satisfy requirement (2) for the same reason that the SPV proof design does not satisfy requirement (2). It is a fundamental feature of bitcoin that 51% attackers are able to control the contents of the blockchain _absolutely_ from the moment they gain the hashpower majority, and so it is a given that they can prevent a transaction from being confirmed until an "honest" majority regains control. So in practice requirement (2) is impossible to satisfy so long as bitcoin relies on Nakamoto consensus for security and transaction withholding goes unpunished.

Another protocol that satisfies requirement (1) is a validity sidechain. This is a variety of pegged sidechain described in Section 6.1 of the pegged sidechains whitepaper:

> An exciting recent development in academic cryptography has been the invention of SNARKs. SNARKs are space-efficient, quickly verifiable zero-knowledge cryptographic proofs that some computation was done... Then blocks could be constructed which prove their changes to the unspent-output set, but do so in zero-knowledge in the actual transactions. They could even commit to the full verification of all previous blocks, allowing new users to get up to speed by verifying only the single latest block. These proofs could also replace the DMMSes used to move coins from another chain by proving that the sending chain is valid according to some rules previously defined. [ibid]

The first production implementation of a validity sidechain protocol is Zendoo, which was activated on the Horizen mainnet in December 2021.[129] The Zendoo protocol works by requiring that sidechains are registered on the parent chain and then submit "withdrawal certificates" as checkpoints on the parent chain at the end of each sidechain "epoch". Each epoch consists of some number of sidechain blocks that have been added to the chain tip since the end of the previous epoch, plus the forward and backward transfer requests to and from the sidechain. The withdrawal certificate contains data about the sidechain, such as the sidechain ID, the epoch ID, the quality of the certificate (showing that it represents the state of the canonical chain), a hash of the current state of the sidechain, a hash of the last sidechain block accounted for, a bit vector defining all of the UTXOs that have been modified (consumed and created), and a zk proof that proves that the withdrawal certificate is valid according to both the rules of the sidechain and the Zendoo protocol implemented on the parent chain.

A problem can arise if the majority of sidechain block producers continue to produce blocks for an epoch but they do not share the data for these blocks with anyone else, and at they end of the epoch they confirm a withdrawal certificate for this epoch. This can create a data availability problem. If the sidechain experiences this data withholding fault, then any user who had a transaction confirmed in the unavailable block(s) and _does not_ have the data for the transaction will lose access to the UTXO(s) involved until the data becomes available. However, using the information in the withdrawal certificate confirmed on the parent chain, all users who _do_ have a copy of the transaction in which they received a UTXO in the current UTXO set (even just a copy of the raw transaction broadcast over the p2p network) will have enough data to produce a zk proof that proves they own the UTXO as of the last confirmed sidechain state. The zk proof can then be used to unilaterally exit the sidechain back to the parent chain.

The Zendoo protocol accepts a nuanced tradeoff between data availability and scalability. Recall that in each state update confirmed in the parent chain blocks, validity rollups store all of the transaction data that users need to produce their own validity proofs and unilaterally exit back to the parent chain. Validity rollups on bitcoin could therefore confirm at most around 250,000 transactions in bitcoin's 4 million WU block size limit (see details in Table 2). In contrast, Zendoo stores a highly compressed representation of the current state of the sidechain in parent chain blocks. This compressed representation of the sidechain state puts some users at a risk of data unavailability and imposes a limit on the number of UTXOs that can exist on the sidechain. This puts Zendoo somewhere between validity rollups and validia chains in terms of the strength of its ownership security and its scalability potential.

The current implementation of Zendoo has a fixed cost of around 152 kilobytes (KB) per withdrawal certificate, assuming a zk proof size of about 45 KB, a bit vector size of about 106 KB (though this varies based on the parameters of the bit vector), and 1 KB for the other miscellaneous sidechain information included in the certificate. The 106 KB bit vector is parameterized to store a maximum of 4,000,000 sidechain UTXOs and 100,000 UTXO updates per epoch. This means that 100,000 transactions could occur on the Zendoo sidechain and so long as the sidechain UTXO set never contains more than 4 million UTXOs, the cost in terms of parent chain block space will be 152 KB per epoch.

**Table 7. Comparing the per-block capacity of different types of UTXO model dual-funded Lightning channel open transactions**

| Transaction type             | 1,000,000 WU                          | 3,000,0000 WU                       | Total (tx/block) |
| ---------------------------- |:-------------------------------------:|:-----------------------------------:|:----------------:|
| Rollup 1-input-2-output UTXO | Validity proof, script, other tx data | 26,432 txs (113.5 WU/tx)            | 26,432           |
| Zendoo 1-input-2-output UTXO | Validity proof, script, other tx data | 100,000 txs (106,000 WU/bit vector) | 100,000          |

> Table 7 description: The block is broken up into 1,000,000 WU and 3,000,000 WU portions to illustrate how in the case of rollup transactions, some of the weight limit is reserved for the validity proof and script data while the rest can be used for the rollup transaction data. In the case of Zendoo state updates, there is also a fixed for the validity proof and script data, and Zendoo state updates currently max out at around 100,000 txs per bit vector costing at most 106,000 WU per withdrawal certificate (assuming that, similar to validity rollup data as described in Section 4.1, the witness discount is applied to Zendoo bit vector data).

## Appendix E. Enabling additional throughput increases with validity proofs

Validity proofs can be used to help bitcoin scale even further than validity rollups alone enable. Using recursive proofs, the validity of the blockchain up to a given block height could be proven using a single aggregated proof.[130] The implication is that the block size limit could be increased without making it any harder for full nodes to verify the blockchain. The block size limit wouldn't be able to be increased _too much_, and developers would still need to ensure block propagation times remain reasonable, but even a 50-100% increase could mean up to an additional 125,000-250,000 rollup transactions per block. 

There are several factors holding back bitcoin block size limit increases today. One is that the larger blocks are allowed to get, the longer they take to verify.[131] A uniquely formatted block could be easy for a powerful computer to verify, but harder for weaker machines to verify. This puts more powerful computers at a mining advantage, because they can mine one of these uniquely formatted blocks, broadcast it to the network, and begin working on the next block before less powerful miners have finished verifying the block. Related is the time it takes for initial block download (IBD). As larger blocks are added to the chain tip, it takes longer and longer for full nodes to sync up to the chain tip and then keep up with the chain tip as new blocks are mined.

Another factor holding back block size limit increases is block propagation time due to network bandwidth constraints. Larger blocks propagate slower through the network by virtue of their larger size relative to the bandwidth speeds available to peers on the network.[ibid] This effect can create an incentive for centralization of mining in private, high-speed networks, or even centralization within the same data centers, so that miners can quickly propagate blocks to each other without the latency introduced by p2p network bandwidth constraints.

Proofs of the validity of the blockchain solve the IBD issue for blocks of arbitrary size by removing the requirement that full nodes download and execute every transaction in every block to sync up to the chain tip. Instead, a validity proof can be produced as of a given block height `n` that can be used to convince full nodes that 1) a copy of the UTXO set is derived from the heaviest valid chain as of block `n` and 2) that newly received blocks `n+1, n+2, etc` correctly build upon the heaviest valid chain. This means that full nodes only have to download a copy of the UTXO set as of block `n`, verify the proof of `n`, and execute transactions in blocks built on top of `n` up until the next proof, rather than execute all transactions in all blocks from genesis. Proofs can continue to be generated for each block or for batches of blocks to maintain fast IBD and sync times for full nodes.

This "proof-sync" technique may solve the IBD problem of large blocks, but it does not solve the mining centralization problems. Work still needs to be done to decrease the bandwidth costs and propogation times of increased Layer 1 transaction volume, for example by implementing or further optimizing protocols such as Compact Block Relay and Erlay that reduce bandwidth requirements even as transaction volume increases.[132, 133] And there's no getting around the requirement for storage of blockchain data, which increases as the size of the blockchain grows. Still, having one less problem to solve (the IBD problem) is a nice improvement. The best part is that no consensus rule changes are required to implement and start using proof-sync clients. As of the time of writing, Lukas George and Robin Linus have begun work on a proof-sync bitcoin client implementation written in Cairo called "Zerosync".[134] 

## References

[1] https://bitcointalk.org/index.php?topic=770.msg8637#msg8637

[2] https://people.csail.mit.edu/silvio/Selected%20Scientific%20Papers/Proof%20Systems/The_Knowledge_Complexity_Of_Interactive_Proof_Systems.pdf

[3] https://en.bitcoin.it/wiki/Zero_Knowledge_Contingent_Payment

[4] https://bitcoincore.org/en/2016/02/26/zero-knowledge-contingent-payments-announcement/

[5] https://zerocoin.org/media/pdf/ZerocoinOakland.pdf

[6] https://yewtu.be/watch?v=Q4nWoEKUtgU

[7] https://explorer.firo.org/block/c0c53331e3d96dbe4a20976196c0a214124bef9a7829df574f00f4e5a1b7ae52

[8] https://electriccoin.co/blog/zcash-sprout-launch/

[9] https://medium.com/starkware/the-cambrian-explosion-of-crypto-proofs-7ac080ac9aed

[10] https://en.bitcoin.it/wiki/Block_size_limit_controversy#Damage_to_decentralization

[11] https://blog.lopp.net/bitcoin-core-performance-evolution/

[12] https://www.researchgate.net/publication/340300470_Design_Patterns_for_Gas_Optimization_in_Ethereum

[13] https://blog.ethereum.org/2014/10/21/scalability-part-2-hypercubes/

[14] https://blog.zilliqa.com/zilliqa-mainnet-the-launch-and-beyond-4cd7e113369f/

[15] https://members.delphidigital.io/reports/the-hitchhikers-guide-to-ethereum

[16] https://mimblewimble.cash/

[17] https://www.plan99.net/~mike/satoshi-emails/thread4.html

[18] https://bitcointalk.org/index.php?topic=244656.0

[19] https://lightning.network/lightning-network-paper.pdf

[20] https://www.bbc.com/news/technology-42237162

[21] https://medium.com/web3foundation/scalingnow-scaling-solution-summit-summary-be30047047bf

[22] https://medium.com/giveth/tackling-ethereum-scalability-issues-29bd700b5060

[23] https://medium.com/poa-network/poa-bridge-launch-live-demo-e3e7c9e4f08

[24] https://vitalik.ca/general/2019/06/12/plasma_vs_sharding.html

[25] https://www.ibtimes.co.uk/ethereums-vitalik-buterin-explains-how-state-channels-address-privacy-scalability-1566068

[26] https://medium.com/statechannels/counterfactual-generalized-state-channels-on-ethereum-d38a36d25fc6

[27] http://plasma.io/plasma-deprecated.pdf

[28] https://ethereum.org/en/developers/docs/scaling/plasma/#the-mass-exit-problem-in-plasma

[29] https://github.com/ethereum/research/wiki/A-note-on-data-availability-and-erasure-coding

[30] https://ethresear.ch/t/plasma-cash-plasma-with-much-less-per-user-data-checking/1298

[31] https://medium.com/@kelvinfichter/why-is-evm-on-plasma-hard-bf2d99c48df7

[32] https://medium.com/plasma-group/plapps-and-predicates-understanding-the-generalized-plasma-architecture-fc171b25741

[33] https://vitalik.ca/general/2019/08/28/hybrid_layer_2.html

[34] https://github.com/barryWhiteHat/roll_up

[35] https://twitter.com/EliBenSasson/status/1450173650041245699

[36] https://twitter.com/fuellabs_/status/1344707195250896899

[37] https://medium.loopring.io/loopring-deployed-protocol-3-0-on-ethereum-a33103c9e5bf

[38] https://medium.com/aztec-protocol/launching-aztec-2-0-rollup-ac7db8012f4b

[39] https://medium.com/ethereum-optimism/introducing-evm-equivalence-5c2021deb306

[40] https://vitalik.ca/general/2022/08/04/zkevm.html

[41] https://twitter.com/krzKaczor/status/1524753291912962048

[42] https://twitter.com/EdanYago/status/1527808313689354241

[43] https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698

[44] https://research-development.nomadic-labs.com/tezos-is-scaling.html

[45] https://jsidhu.medium.com/blockchain-idealisms-b61c5781ddc3

[46] https://blog.celestia.org/sovereign-rollup-chains/

[47] https://hrf.org/zkrollups

[48] https://medium.com/starkware/redefining-scalability-5aa11ffc5880 

[49] https://medium.com/@deaneigenmann/optimistic-contracts-fb75efa7ca84

[50] https://polynya.medium.com/updated-thoughts-on-modular-blockchains-ce1b159fa1b3

[51] https://blog.matter-labs.io/introducing-zk-sync-the-missing-link-to-mass-adoption-of-ethereum-14c9cea83f58

[52] https://blog.celestia.org/clusters/

[53] https://blockstream.com/sidechains.pdf 

[54] https://twitter.com/ercwl/status/1311452208127447044

[55] https://twitter.com/statelayer/status/1311851018741846017

[56] https://twitter.com/ercwl/status/1497801246295605250

[57] https://twitter.com/ercwl/status/1497966811928801282

[58] https://twitter.com/hodlonaut/status/1086703428791865345

[59] https://twitter.com/hodlonaut/status/1091987050675490818

[60] https://twitter.com/ercwl/status/1497307148317044741

[61] https://bitcointalk.org/index.php?topic=1790

[62] https://counterparty.io/docs/faq-smartcontracts/#can-ethereum-smart-contracts-run-on-counterparty

[63] https://medium.com/iovlabs-innovation-stories/bitcoin-sidechains-74a72ceba35d

[64] https://lightco.in/2020/08/02/bitcoin-pegs/

[65] https://blog.rsk.co/noticia/blockchain-bridges-an-industry-overview/

[66] https://yewtu.be/watch?v=9s3EbSKDA3o

[67] https://bitcoinmagazine.com/technical/state-of-bitcoin-lightning-network-privacy

[68] https://medium.com/boltlabs/zkchannels-for-bitcoin-f1bbf6e3570e

[69] https://bitcoiner.guide/privacy/

[70] https://medium.com/aztec-protocol/introducing-private-bitcoin-1cd9d895c770

[71] https://blog.blockstream.com/blockstream-sponsors-federated-e-cash-as-a-bitcoin-scaling-technology/

[72] http://zerocash-project.org/paper

[73] https://explorer.zcha.in/transactions/169b903466119c78653eaf94d8d4b69eb51c38a3a6fcaa318f21f41adcec59ea

[74] https://medium.com/aztec-protocol/launching-aztec-2-0-rollup-ac7db8012f4b

[75] https://medium.com/aztec-protocol/aztecs-zk-zk-rollup-looking-behind-the-cryptocurtain-2b8af1fca619

[76] https://explorer.aztec.network/tx/994d892ebc203949e632add016796f69a88a42d3f4a40a58cae60c09ecc3e46d

[77] https://twitter.com/_prestwich/status/1284174486674083840

[78] https://vitalik.ca/general/2021/01/05/rollup.html

[79] https://bitcoinops.org/en/tools/calc-size/

[80] https://en.bitcoin.it/wiki/Weight_units

[81] https://crypto.stackexchange.com/a/6375

[82] https://medium.com/starkware/fractal-scaling-from-l2-to-l3-7fe238ecfb4f

[83] https://eprint.iacr.org/2019/1021.pdf

[84] https://medium.com/starkware/recursive-starks-78f8dd401025

[85] https://lightning.network/lightning-network-paper.pdf

[86] https://blog.blockstream.com/c-lightning-opens-first-dual-funded-mainnet-lightning-channel/

[87] https://twitter.com/JohnCantrell97/status/1478794694159220747

[88] https://lists.linuxfoundation.org/pipermail/lightning-dev/2018-December/001752.html

[89] https://tr3y.io/articles/crypto/bitcoin-zk-rollups.html

[90] https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019885.html

[91] https://blockstream.com/simplicity.pdf

[92] https://dusk.network/news/zero-knowledge-plonk-demo

[93] https://twitter.com/EliBenSasson/status/1275423886788694018

[94] https://en.bitcoin.it/wiki/Scalability#CPU

[95] https://bitcoin.stackexchange.com/a/60008

[96] https://bitcoinist.com/double-spend-fud-crashes-bitcoin-below-30000-return-of-bear-trend/

[97]  https://web.archive.org/web/20211026122447/https://www.coindesk.com/markets/2014/01/09/bitcoin-miners-ditch-ghashio-pool-over-fears-of-51-attack/

[98] https://www.erisian.com.au/wordpress/2016/01/07/bitcoin-fees-in-history

[99] https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-December/015455.html

[100] https://arxiv.org/abs/1904.05234v1

[101] https://hackmd.io/ivUzk3piQEG8ALzCGbxlag#311-Fair-ordering

[102] https://hackmd.io/ivUzk3piQEG8ALzCGbxlag#312-Privacy-solutions

[103] https://blog.bitmex.com/txwithhold-smart-contracts/

[104] https://github.com/0xbunnygirl/request-for-reorg

[105] https://github.com/DZGoldman/Deorg

[106] https://eprint.iacr.org/2018/581.pdf

[107] https://www.truthcoin.info/blog/drivechain/

[108] https://medium.com/@deaneigenmann/optimistic-contracts-fb75efa7ca84

[109] https://lightco.in/2022/06/15/miners-can-steal-2/

[110] https://www.pymnts.com/cryptocurrency/2022/privacy-coin-moneros-use-in-ransomware-fuels-growing-security-concerns/

[111] https://fc18.ifca.ai/preproceedings/6.pdf

[112] https://twitter.com/allenyhsu/status/1203879946591981568

[113] https://tradelayer.substack.com/p/trade-offs-in-zk-design-space

[114] https://en.bitcoin.it/wiki/Privacy#Amount_correlation

[115] https://en.bitcoin.it/wiki/Privacy#Change_address_detection

[116] https://abytesjourney.com/lightning-privacy/

[117] https://docs.grin.mw/about-grin/privacy/

[118] https://docs.beam.mw/Lelantus-MW.pdf

[119] https://bytecoin.org/old/whitepaper.pdf

[120] https://www.getmonero.org/2016/09/19/monero-0.10.0-released.html

[121] https://www.getmonero.org/2018/10/11/monero-0.13.0-released.html

[122] https://www.exploremonero.com/transaction/9942cb34786bbb46f106ed31a6bb67eeb6e946d592def0900cab60ca83199f22

[123] https://z.cash/technology/zksnarks/

[124] https://twitter.com/socrates1024/status/1224434578149888000

[125] https://medium.com/starkware/volition-and-the-emerging-data-availability-spectrum-87e8bfa09bb

[126] https://blog.matter-labs.io/zkporter-a-breakthrough-in-l2-scaling-ed5e48842fbf

[127] https://blog.celestia.org/celestiums/

[128] https://ethresear.ch/t/adamantium-power-users/9600

[129] https://blog.horizen.io/horizens-cross-chain-protocol-zendoo-is-live-on-mainnet-build-on-the-zero-knowledge-network-of-blockchains/

[130] https://eprint.iacr.org/2020/352.pdf

[131] https://bitfury.com/content/downloads/block-size-1.1.1.pdf

[132] https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki

[133] https://arxiv.org/abs/1905.10518

[134] https://github.com/lucidLuckylee/zerosync
