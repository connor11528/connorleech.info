---
layout: post
title: Some basics about Stellar and cryptocurrency
date: "2015-10-22"
category:
lede: "Here's to doing some research about non federated networks and financial systems for radical inclusion. Olé!"
author: Connor Leech
published: true
---

Learn you cryptocurrency. Stellar is a new network protocol shaking up the scene

[Stellar](https://www.stellar.org/) is a new protocol that makes it easy to send money between any two currencies quickly. The system aims to bring the world together by increasing interoperability between diverse financial systems and currencies. Parties can exchange using their own defined currencies, currencies that exist already or use the Stellar intermediary currency to perform trades. Stellar is a cryptocurrency and a protocol. The system is managed by a non-profit company based in San Francisco. Stellar is a peer to peer protocol that enables money to move as easily as email. 

I am new to cryptocurrency and have no bitcoin. I signed up with Facebook and recieved free stellars out of interest on the topic. I learned about the Stellar protocol through a post on the [Praekelt Foundation's blog](http://blog.praekeltfoundation.org/).

There they say

> More recently, with the use of Stellar’s open-source protocol, Praekelt is working on a mobile wallet that allows people to save cash or airtime using Vumi, their messaging platform—think Whatsapp, but open source and designed for the developing world, with a focus on improving the economic security of girls in South Africa.

The Praekelt Foundation is a South African company that builds awesome services out of Python and Javascript for the developing world. They also partnered with Facebook et al's [internet.org](https://internet.org/) initiative to bring internet services and connectivity to the rest of the planet. They made a way for Indian's to [access Wikipedia without internet](http://blog.praekeltfoundation.org/post/65981723628/wikipedia-zero-over-text-with-praekelt-foundation). And also made election monitoring software for Libya and ran a successful Guiness advertising campaign in Nigeria. Their [Vumi platform](http://vumi.org/case-studies/) makes it possible to connect acoss modern channels to people that use crappy phones only capable of SMS and USSD messaging. They are very clever so when I saw they were using Stellar I began to pay attention.

There are three things that make the [Stellar Consensus Protocol (SCP)](https://medium.com/a-stellar-journey/on-worldwide-consensus-359e9eb3e949) unique. There is no gatekeeper, there is a low barrier to get started in the network and the system is robust in the face of failure. These properties are called decentralized control, low latency, flexible trust, and asymptotic security. A professor at Stanford, who now works for Stellar, wrote a white paper on the topic [here](https://www.stellar.org/papers/stellar-consensus-protocol.pdf). The gist is that nodes in the network trust other nodes and through this network of trust transactions happen. There is no proof of work model or mining like in the bitcoin system. The stellar protocol, like other cryptocurrencies is [open source](https://github.com/stellar/).

Stellar solves a problem in computer science called the Byzantine Generals problem. I pretty much quoted that from [Marc Andreesen's NYTimes bitcoin post](http://dealbook.nytimes.com/2014/01/21/why-bitcoin-matters/?_r=0) though stellar faces the same issue. The issue is how to know when a transaction is valid when people are trying to scam you all over the place and the network ie the internet is mad insecure.

Bitcoin solves this problem through a system called "proof of work" to ensure valid transactions occur. In the bitcoin system capitalists operate powerful machines called bitcoin miners that perform algorithmic calculations and difficult math problems to update the bitcoin blockchain, which acts as a ledger of all past transactions. After a couple of hours or more of computationally intensive work the blockchain gets updated with a permanent transaction history of everything that occured on the bitcoin network. The owners of the bitcoin miners make a profit, in bitcoin. The world's supply of bitcoin is fixed. To exchange bitcoin for "normal people money"TM one must use a bitcoin exchange. Some of these exchanges are quite shady. For example, Mt. Gox, a popular bitcoin exchange conveniently misplaced, lost, or was robbed of millions of dollars worth of bitcoin. Other bitcoin exchanges, like Coinbase, are entirely legitimate.

Stellar is more of a protocol than a curency. You can use Stellar to exchange dollars for bitcoins or bitcoin for yen. You could also use the stellar protocol to exchange yen for dollars or vice versa. The idea behind stellar is to use it in the background. Email and the internet use all sorts of protocols. Most people that use the internet and email know nothing about the underlying technology.


Stellar, like bitcoin, operates a decentralized network, there is a ledger of historical transactions and both systems have their own currencies. However, the stellar system works entirely differnetly than the bitcoin system. The stellar network is built on trust between members of the network. Anyone can join the stellar network and choose who to trust. Many independent servers participate in the stellar network, so the network can survive even if it loses one or many of its network nodes. The network will still succeed even if some servers fail.

There is no way to mine stellar. The stellar network is made of stellar nodes. Participants in the system choose who to trust on the network. The people you choose to trust on the network are called a quorum slice. A quorum slice can convice one node that a transaction is valid. SCP takes a ballot-based approach to the problem. Quorum slices overlap to create a quorum. A quorum convinces the whole network that a transaction is valid. A quorum is comprised of overlapping quorum slices. Overlapping quorum are called quorum intersections. Once a quorum is reached and the network is in agreement, the ledger on every node in the network is updated. The transaction occurs. The whole process of agreement between all nodes in the network only takes between two and five seconds. The stellar protocol is powerful in that trades can happen between parties who don’t trust each other. The SCP network reaches consensus when all nodes update their ledgers and externalize the same values.

There is a lot of things about the protocol I do not understand but I think it is cool and will keep my eye on this technology moving forward.

#### Resources

To use the Stellar API, [this](http://gdb.svbtle.com/the-stellar-domain-model) is a good conceptual introduction for programmers.

Stellar has [learning resources here](https://www.stellar.org/developers/)

There is the [academic white paper](https://www.stellar.org/papers/stellar-consensus-protocol.pdf
) and [summary of that white paper](https://medium.com/a-stellar-journey/on-worldwide-consensus-359e9eb3e949)

There is also a post about Stellar.. [for Harry Potter fans](http://www.kalzumeus.com/2014/08/05/harry-potter-and-the-cryptocurrency-of-stars/)

![](http://media.giphy.com/media/HIqV5khg6gyqc/giphy.gif)


Stellar also has a [public slack channel](http://slack.stellar.org/) for answering questions and collaborating.