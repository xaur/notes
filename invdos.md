# Notes on Decred and invdos vulnerability

## Links

- https://github.com/advisories/GHSA-hx3r-jv9q-85jw
- https://nvd.nist.gov/vuln/detail/CVE-2018-17145
- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-17145
- https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures#CVE-2018-17145

## Media coverage

- Sep-09 https://www.coindesk.com/high-severity-bug-in-bitcoin-software-revealed-2-years-after-fix
- Sep-12 https://www.zdnet.com/article/researcher-kept-a-major-bitcoin-bug-secret-for-two-years-to-prevent-attacks/
- Sep-12 https://securityaffairs.co/wordpress/108188/hacking/invdos-dos-bitcoin-core.html
- Sep-13 https://decrypt.co/41685/developers-reveal-2018-bitcoin-bug-used-to-crash-entire-networks
- Sep-14 https://thedailychain.com/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains/
- Sep-14 https://www.cryptoglobe.com/latest/2020/09/bitcoin-engineers-rediscover-major-blockchain-vulnerability-on-decred/
- Sep-17 https://thedailychain.com/decred-clarify-details-of-reported-vulnerability/

## Discussions

- https://matrix.to/#/!zefvTnlxYHPKvJMThI:decred.org/$MFVmkimdCIH80kM-OcVSGI4LRBSXexVXB9O_TM-7msY
- https://www.reddit.com/r/decred/comments/isifn2/bitcoin_engineers_identify_and_patch/

## Reflection

The good bits:

- most importantly, the vuln was submitted and patched
- this is debatable, but the time given to patch and upgrade was _somewhat_ reasonable (64 days between original submission and the public disclosure via CoinDesk)
- the researchers have notified Decred about their intent to disclose publicly 13 days in advance (51 day since the original submission)
- bug bounty program has proven to work and incentivize security research of Decred software
- Decred and bug bounty got some media mentions (ZDNet, Cryptoglobe)
- The Daily Chain posted an update clarifying the risks with a quote from @davecgh

The bad bits:

- researchers' plan to go public in early Sep was unexpected by Decred and the devs had to cut an unplanned patch release
- operators of Decred nodes who build from source had 63 days to upgrade between the patch on master and public disclosure, those who use binaries had 13 days to upgrade, assuming all have been perfectly notified at the moment of master patch/binary release
- 64 days is not "responsible disclosure" if we use Google Project Zero's 90-day delay as a standard
- Decred got some negative press from all the outlets that mixed up the facts (all the way down to claiming that "Bitcoin devs patched a vulnerability Decred")
- this disclosure and media push raise too many questions, e.g.
  - why btcd an dcrd are the only two projects chosen by the paper that have not been patched back in 2018
  - why Sep 9
  - why not wait a bit more until Decred has upgraded enough (after all, it was a subject of the paper and it paid a bounty)
  - why not check and notify bigger coins like Dash
  - why so much misinformation

## Media observations

Delays between some timestamps suggest it was a professional media push whose organizers had fast rails to outlets like CoinDesk and ZDNet.

Some articles and tweets (mostly the early ones) do not have misinformation about Decred and do not focus on it at all (like CoinDesk). All they do is help you imagine how bad it _could_ go, while researchers themselves acknowledge there is no trace of it being exploited in the wild.

My non-malicious theory is that this media push is to make the media presence and promotion for researchers who found the vulnerability, and possibly the Bcoin project. My main argument in favor of this theory is the observation that CoinDesk article was published 45 min after the paper PDF was updated on the invdos.net webserver, and another observation that the CoinDesk release was made 0.5h _before_ the bitcoin-dev mailing list was notified. The "official" version why the vuln was only disclosed 2 years after it was discovered and patched in Bitcoin, is because the network was not "safe enough" to disclose (which implies it has only become "safe" in Sep 2020).

But some articles and tweets contain outright misinformation, down to the blatant headline "Bitcoin engineers identify and patch vulnerability in Decred, Btcd blockchains" in The Daily Chain. My non-malicious theory here is that, observing that misinformation mostly comes from "second wave" media, the "journalists" did their best to craft clickable headlines while not wasting too much time on due research.

## Misinformation

tl;dr the misinformation was nicely addressed in [this](https://twitter.com/degeri_crypto/status/1305591774698786821) short thread by @degeri.

[ZDNet](https://www.zdnet.com/article/researcher-kept-a-major-bitcoin-bug-secret-for-two-years-to-prevent-attacks/):

> Technical details were published earlier this week after the same vulnerability was independently discovered in another cryptocurrency, based on an older version of the Bitcoin code that hadn't received the patch.

Severity = low. This para does not name the coin but I'm sure it refers to Decred. If so, this reads like "Decred is based on Bitcoin code", which is technically correct but puts Decred in the same bucket with low effort forks of Bitcoin Core. See [Decred is not a fork of Bitcoin](https://github.com/decredcommunity/wiki/blob/master/wiki/misconceptions.md#decred-is-a-fork-of-bitcoin) for details.

[Decrypt](https://decrypt.co/41685/developers-reveal-2018-bitcoin-bug-used-to-crash-entire-networks):

> Bitcoin Engineers Rediscover Huge Blockchain Vulnerability

Saying that "blockchain" was vulnerable overblows the issue. The vuln (theoretically) allows to crash the nodes and slow down the network, but the "blockchain" (the database, cryptography, coins) is not compromised by it.

> could have led to entire systems of nodes being shut down

Overblows the issue. The shut down nodes can be restarted. It is a question of how long it takes to crash one node vs how long it takes to restart it. Please contact experts for more details.

> published a research paper this week detailing how they found it in a number of other blockchain iterations: Btcd and Decred.

btcd (lowercase) is not a blockchain iteration. It is an alternative full node implementation for Bitcoin blockchain. Bitcoin Core, Bcoin, btcd and others use the same blockchain.

> A month later, Khan discovered the vulnerability in another blockchain network, Decred. Khan, in tandem with other blockchain engineers, rolled out fixes to the vulnerabilities in late August.

Decred was fixed by David Hill in pull request [2253](https://github.com/decred/dcrd/pull/2253). Khan did fix btcd in [1599](https://github.com/btcsuite/btcd/pull/1599) and [1603](https://github.com/btcsuite/btcd/pull/1603), and note that both used code from Decred. I have to point this out because it appears that this imprecise wording has led The Daily Chain (which referenced this Decrypt story) to believe that "Bitcoin developers patched Decred", which is false.

[The Daily Chain](https://thedailychain.com/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains/):

> Bitcoin engineers identify and patch vulnerability in Decred, Btcd blockchains

Double false. Decred was [patched](https://github.com/decred/dcrd/pull/2253) by Decred devs. btcd is not a blockchain.

> the vulnerability would allow an attacker to essentially shut down the blockchains in question

Slow down, but not shut down. Please contact experts for clarifications.

## Media timeline

- utc: 2020-07-07
- event: On 2020-Jul-07 we were notified via our bounty program about of this vulnerability in Decred by @tuxcanfly. We acknowledged the email within 2 hours. We investigated and opened a PR to patch the vulnerability within a day https://github.com/decred/dcrd/pull/2253
- source_url: https://twitter.com/degeri_crypto/status/1305591778578493440
- note: time unknown

---

- utc: 2020-07-07 14:38:27
- event: tuxcanfly submitted PR 1599 to btcd
- url: https://github.com/btcsuite/btcd/pull/1599
- quote: This PR updates sentNonces and knownInventory to use the more generic LRU from Decred. This cuts down the need for maintaining two instances of almost same code i.e mruInventoryMap and mruSentNonces.
- note: this PR is referenced in the paper as the fix for btcd, although the paper does not mention that it uses code from Decred

---

- utc: 2020-07-08 15:10:14
- event: dhill submitted PR 2253 to fix the vuln in dcrd
- url: https://github.com/decred/dcrd/pull/2253

---

- utc: 2020-07-08 20:44:05
- event: PR 1599 merged to btcd
- url: https://github.com/btcsuite/btcd/pull/1599#event-3526958018

---

- utc: 2020-07-08 23:44:48
- event: PR 2253 merged in dcrd
- url: https://github.com/decred/dcrd/pull/2253#event-3527418323

---

- utc: 2020-07-12 07:04:10
- event: tuxcanfly submitted PR 1603 to btcd
- url: https://github.com/btcsuite/btcd/pull/1603
- quote: Also, increase ban score on notfound and return whether addBanScore disconnects the peer. Backport from decred/dcrd#2253
- note: this PR is not mentioned in the paper, but PR description says that it is a backport of dcrd 2253, which is mentioned in the paper as the fix for dcrd

---

- utc: 2020-07-28 13:23:36
- event: PR 1603 merged in btcd
- url: https://github.com/btcsuite/btcd/pull/1603#event-3594335220

---

- utc: 2020-08-21
- event: Decred bounty was paid as part of the payments for July invoices
- source_url: https://twitter.com/degeri_crypto/status/1305591786065330176

---

- utc: 2020-08-27
- event: our lead developer @davecgh was contacted by both @tuxcanfly and Braydon Fuller (Original finder) that they are going to go public with the CVE
- source_url: https://twitter.com/degeri_crypto/status/1305591786065330176
- note: this is 51th day since the initial submission on 2020-07-07

---

- utc: 2020-08-28 18:41:38
- event: Decred release v1.5.2 on GitHub
- url: https://github.com/decred/decred-binaries/releases/tag/v1.5.2

---

- utc: 2020-08-28 21:09:37
- event: decredproject tweeted about the v1.5.2 patch release with the fix
- url: https://twitter.com/decredproject/status/1299454110559866880

---

- utc: 2020-09-03 20:51:21
- event: domain invdos.net registered via NameCheap for 1 year
- url: https://who.is/whois/invdos.net

---

- utc: 2020-09-09 12:15:27
- event: the paper has HTTP Last-Modified header as "Wed, 09 Sep 2020 12:15:27 GMT", it may be the time when it was uploaded
- url: https://invdos.net/paper/CVE-2018-17145.pdf

---

- utc: 2020-09-09 13:00:00
- event: William Foxley published on CoinDesk
- url: https://www.coindesk.com/high-severity-bug-in-bitcoin-software-revealed-2-years-after-fix
- title: ‘High’ Severity Bug in Bitcoin Software Revealed 2 Years After Fix
- quotes:
  - The vulnerability could not be disclosed publicly for over two years as node operators took longer than expected to update, Fuller said.
- notes:
  - Khan gave comments to CoinDesk
  - note that the story published timestamp is just 45 minutes since Last-Modified of the PDF
  - this is 2^8th day since Decred was notified of the vuln :tada:. Note that 64 is less than the 90 days disclosure delay adopted by Google Project Zero (if can call this a "standard", not sure)
  - 'High' in quotes of the title may indicate the professionalism of the journalist
  - it carefully does not mention word "Decred" at all (as usual for CoinDesk), despite it was one of the few projects analyzed by the paper
  - it does not mention that Khan was hunting for bugs in Decred for its bug bounty program, like ZDNet did

---

- utc: 2020-09-09 13:01:10
- event: CoinDesk tweeted
- url: https://twitter.com/CoinDesk/status/1303679841296842753
- quote: "JUST IN: A previously undisclosed bug in @bitcoincoreorg software could have let attackers steal $BTC, delay transfers or split the network had it not been patched in 2018. @wsfoxley reports trib.al/X3tRUco"
- note: several replies think it is a well timed FUD for BTC

---

- utc: 2020-09-09 13:28:38
- event: Braydon Fuller posted in bitcoin-dev mailing list
- url: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-September/018164.html
- notes:
  - the fact that Bitcoin developers mailing list was notified 0.5h _after_ the CoinDesk release is the main reason why I believe the main goal of this disclosure was self-promotion
  - zero replies in that thread as of 2020-09-15 (6 days passed) hints at the level of interest Bitcoin devs had in this vulnerability https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-September/thread.html 
  - referenced by Bitcoin Optech newsletter to be released Sep 16 https://github.com/bitcoinops/bitcoinops.github.io/pull/458

---

- utc: 2020-09-09 13:55:15
- event: @Bcoin replied to "why did it take a couple years to publicly disclose it?"
- url: https://twitter.com/Bcoin/status/1303693454749380611
- quote: Because the engineers were waiting for enough patched clients to be deployed. The CVE was responsibly disclosed as soon as it was discovered.

---

- utc: 2020-09-09 13:57:07
- event: @Bcoin tweeted CoinDesk story
- url: https://twitter.com/Bcoin/status/1303693925077655552
- quote: "Vulnerability discovery and responsible disclosure by bcoin developers from almost two years ago: The network is finally safe enough for public reports."
- linked_url: https://twitter.com/CoinDesk/status/1303679841296842753

---

- utc: 2020-09-09 13:58:22
- event: @Bcoin tweeted the invdos website
- url: https://twitter.com/Bcoin/status/1303694236978745346
- quote: Here is the public disclosure and research paper by bcoin engineers Braydon Fuller and Javed Khan (@tuxcanfly) explaining the vulnerability they discovered and responsibly disclosed over two years ago. https://invdos.net/

---

- utc: 2020-09-09 14:00:09
- event: Matthew Zipkin (Bcoin and Handshake contributor) tweeted
- url: https://twitter.com/MatthewZipkin/status/1303694686826135555
- quote: "This is the benefit of alternative implementations of #Bitcoin: Different perspectives. Sometimes you don't notice something is insecure until you try to re-implement it in another language. I am incredibly proud of the @bcoin team and especially this responsible disclosure."
- linked_url: https://twitter.com/Bcoin/status/1303694236978745346

---

- utc: 2020-09-09 14:51:30
- event: @nobsbitcoin (1.3K) tweeted
- url: https://twitter.com/nobsbitcoin/status/1303707609741041665
- quote: "CVE-2018-17145: Bitcoin Inventory Out-of-Memory Denial-of-Service Attack - Doesn't appear to have been exploited but could have been used to steal funds. - Vuln introduced Nov 2017. - Discovered June 2018. - Bitcoin Core v0.16.2 included fix July 2018. https://invdos.net/paper/CVE-2018-17145.pdf"

---

- utc: 2020-09-09 21:36:31
- event: PastaPastaPasta (Dash Core Group Developer) submitted a pull request with the fix to Dash Core
- url: https://github.com/dashpay/dash/pull/3694
- note: Dash has $730M cap vs $160M of Decred as of 2020-09-14, but post-disclosure PR suggests it was not notified at all

---

- utc: 2020-09-10 17:01:48
- event: CVE-2018-17145 data updated in cvelist repo, status changed from RESERVED to PUBLIC, added info and links to invdos.net
- url: https://github.com/CVEProject/cvelist/commit/0cc7a971840d710cac1a341f25f33805f833e137

---

- utc: 2020-09-10
- event: CVE-2018-17145 published at nvd.nist.gov
- url: https://nvd.nist.gov/vuln/detail/CVE-2018-17145
- note: time unknown

---

- utc: 2020-09-10 15:45:00
- event: CoinDesk piece updated
- url: https://www.coindesk.com/high-severity-bug-in-bitcoin-software-revealed-2-years-after-fix
- quote: Since publication, this article has been updated to include a link to the paper and additional information about one of its co-authors and about the vulnerability it described.

---

- utc: 2020-09-10 19:44:58
- event: braydonf published CVE-2018-17145 in GitHub Advisory Database
- url: https://github.com/advisories/GHSA-hx3r-jv9q-85jw
- note: in the header it notes that the affected bcoin npm package was patched in version 1.0.2, which was tagged 2018-Jul-13

---

- utc: 2020-09-11 18:29:00
- event: modified_time attribute of the CoinDesk story as of 2020-09-14 21:00
- url: https://www.coindesk.com/high-severity-bug-in-bitcoin-software-revealed-2-years-after-fix
- note: the diff since Sep 9 version is insignificant, with most notable change being the addition of the paragraph about the impact on LN, "Layer 2 (L2) solutions such as the Lightning Network (...) were at risk due to the vulnerability. Bitcoin full nodes were not at risk of losing funds."

---

- utc: 2020-09-12 10:25:00
- event: Catalin Cimpanu published in ZDNet's Zero Day blog
- title: Researcher kept a major Bitcoin bug secret for two years to prevent attacks
- url: https://www.zdnet.com/article/researcher-kept-a-major-bitcoin-bug-secret-for-two-years-to-prevent-attacks/
- quotes:
  - quote: Technical details were published earlier this week after the same vulnerability was independently discovered in another cryptocurrency, based on an older version of the Bitcoin code that hadn't received the patch.
  - note: The coin is not named here but I read that this is about Decred.

---

- utc: 2020-09-12 14:18:15
- event: Pierluigi Paganini published in Security Affairs
- url: https://securityaffairs.co/wordpress/108188/hacking/invdos-dos-bitcoin-core.html
- quote: Khan reported the flaw as part of the Decred bug bounty program causing its public disclosure.

---

- utc: 2020-09-12 14:33:55
- event: "PR #3694 merged to Dash Core"
- url: https://github.com/dashpay/dash/pull/3694#event-3759417575

---

- utc: 2020-09-13 18:27:33
- event: Mathew Di Salvo published on Decrypt
- title: Bitcoin Engineers Rediscover Huge Blockchain Vulnerability
- url: https://decrypt.co/41685/developers-reveal-2018-bitcoin-bug-used-to-crash-entire-networks
- quotes:
  - quote: "but published a research paper this week detailing how they found it in a number of other blockchain iterations: Btcd and Decred."
  - quote: "In June 2020, Khan noticed that the old attack applied to Btcd, an alternative Bitcoin blockchain node that doesn’t let its users send or receive payments"
  - note: They put it in an interesting way. I guess it refers to the fact that btcd and btcwallet are separate programs for chain and wallet, where "send or receive payments" is done by btcwallet so btcd cannot do it. But without this clarification, it really reads like btcd is inferior in some way and lacks proper features.
  - quote: Khan, in tandem with other blockchain engineers, rolled out fixes to the vulnerabilities in late August.
  - note: I guess it simply means that people worked together, but I imagine that some will read this as "he patched Decred" (spoiler: The Daily Chain did exactly this :facepalm:)
- note: fly photo as a story image

---

- utc: 2020-09-13 18:28:58
- event: @decryptmedia (20K) tweeted
- url: https://twitter.com/decryptmedia/status/1305211887601283073
- quote: "#Bitcoin Engineers Rediscover Huge Blockchain Vulnerability https://decrypt.co/41685/developers-reveal-2018-bitcoin-bug-used-to-crash-entire-networks"

---

- utc: 2020-09-14 10:05:42
- event: Gareth Jenkinson published on The Daily Chain
- title: Bitcoin engineers identify and patch vulnerability in Decred, Btcd blockchains
- url: https://thedailychain.com/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains/
- quotes:
  - quote: The same threat was then picked up by Khan in June 2020, noting the vulnerability in the Btcd blockchain before research identified that the Decred blockchain was also at risk in July 2020. The latter could have been particularly devastating as 100% of Decred nodes were at risk, as well as nodes serving block filters to Bitcoin Lightning wallets.
- notes:
  - References Decrypt story
  - "Btcd blockchains" :facepalm:

---

- utc: 2020-09-14 10:35:04
- event: u/KarinHernandez submitted to The Daily Chain story to r/decred
- url: https://www.reddit.com/r/decred/comments/isifn2/bitcoin_engineers_identify_and_patch/
- linked_url: https://thedailychain.com/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains/

---

- utc: 2020-09-14 11:00:06
- event: @DailyChainNews (8K) tweeted
- quote: "#Bitcoin engineers have identified and patched a vulnerability in the Decred and Btcd blockchains which was initially picked up in the Bitcoin blockchain back in 2018. https://thedailychain.com/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains/"
- url: https://twitter.com/DailyChainNews/status/1305461313754275841

---

- utc: 2020-09-14 13:59:00
- event: Francisco Memoria published a story in Cryptoglobe
- title: Bitcoin Engineers Rediscover Major Blockchain Vulnerability on Decred
- url: https://www.cryptoglobe.com/latest/2020/09/bitcoin-engineers-rediscover-major-blockchain-vulnerability-on-decred/
- note: References ZDNet story.

---

- utc: 2020-09-14 14:12:06
- event: Saeed Valadbaygi (14K) tweeted
- url: https://twitter.com/SaeedBaygi/status/1305509633906192387
- quote: Bitcoin Engineers Rediscover Major Blockchain Vulnerability on Decred - CryptoGlobe https://www.cryptoglobe.com/latest/2020/09/bitcoin-engineers-rediscover-major-blockchain-vulnerability-on-decred/

---

- utc: 2020-09-14 14:14:11
- event: SurgeWorld (0.3K) tweeted
- url: https://twitter.com/money_whale/status/1305510157372268544
- quote: "#Bitcoin #engineers have rediscovered a major blockchain vulnerability found in 2018 on Bitcoin Core, the software that powers the Bitcoin #blockchain, on the Decred blockchain, which is based on #BTC’s code."

---

- utc: 2020-09-14 16:56:02
- event: u/RambleFeed submitted The Daily Chain to r/decred again, via allyourfeeds.com
- url: https://www.reddit.com/r/decred/comments/isow87/bitcoin_engineers_identify_and_patch/
- linked_url: https://allyourfeeds.com/blockchain/news/bitcoin-engineers-identify-and-patch-vulnerability-in-decred-btcd-blockchains?order=recent

---

- utc: 2020-09-14 19:38:30
- event: degeri tweeted on the misinformation regarding CVE-2018-17145
- url: https://twitter.com/degeri_crypto/status/1305591774698786821

---

- utc: 2020-09-17 16:39:07
- event: Gareth Jenkinson published a clarification on The Daily Chain
- url: https://thedailychain.com/decred-clarify-details-of-reported-vulnerability/
- notes:
  - contains clarification by @davecgh that not 100% of nodes were affected and that Decred was patched by Decred devs
  - 'vulnerability' in quote marks in the title is a nice little bonus
  - thanks to @davecgh and @l1ndseymm for writing and delivering the commentary and for Gareth for posting the update
