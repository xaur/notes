---
published_url: https://github.com/decred/politeia/issues/720
published_utc: 2019-03-11
updated_utc: 2020-04-03
---

`politeiavoter` collects unencrypted wallet private passphrase from the user to sign votes via `dcrwallet`. This post suggests an alternative protocol where sharing passphrase with clients like `politeiavoter` is not necessary. The protocol has a side effect of a step towards enabling signing Politeia votes offline or with hardware signing devices.

It is common in computer security to _not_ share credentials with unprivigeled software. Programs that need to run with elevated privileges do not collect your root password, instead some trusted system component handles password entry and calls the program. Additionally, in mobile operating systems it is common to have a permission system where an app requests a permission to perform a certain operation, the system brings it up to the user, the user approves it temporarily or permanently, and the system grants the app the ability to perform the operation.

My security concern is that `politeiavoter` is a program that communicates with a centralized server over a network. In case of a programming mistake, it could leak the passphrase to the server. This is very unlikely and it would be a shameful epic fail by the developers. Even if that never happens, I'd like to put less trust into `politeiavoter`. I'd like to only share the passphrase with `dcrwallet` and have all other programs that need to sign messages ask `dcrwallet` to do it for them.

A nice property of current system that must be retained is that the wallet is only unlocked for a single call and is immediately locked afterwards. It is not hanging unlocked for any program to (ab)use it, and does not allow malicious clients (that were not given the passphrase, unlike `politeiavoter`) from making inappropiate calls. The authentication must be specific to a single request.

This would require some authentication mechanism where the user confirms to `dcrwallet` that he trusts the request to sign messages. If the user is fooled into approving a malicious request, it could have `dcrwallet` to sign a message which happens to be a transaction that spends user's funds.

The challenge is that `dcrwallet` is a non-interactive command line program. Some mechanism is necessary where `dcrwallet` accepts a signing request from another program, communicates it to the user, collects consent from the user, and sends signed message back to the requesting program.

An example implementation could work as follows:

* user runs `politeiavoter` to cast Politeia votes.
* `politeiavoter` talks to `dcrwallet` in readonly mode and figures out what it needs to sign.
* `politeiavoter` generates a request containing `request id`, sends it to `dcrwallet`, and prints a message instructing the user that he needs to approve the request with that `request id`.
* `dcrwallet` prints the request in the logs (similar to how it prints RPC requests).
* in addition to the logs, the user can manually check for pending requests with `dcrctl --wallet listsignrequests`.
* in both logs and manual check for requests, the printed message must contain enough information for the user to make an informed decision whether to approve it. In a general case, it must show `request id` and pubkeys/addresses for the privkeys that are requested to sign, and the amount of funds they control at the moment. Additional info whether it is a ticket is useful. This should be enough for the user to check and say "ah it's signing stuff with my tickets, that looks ok".
* the user runs `dcrctl --wallet approvesign <request id>`. The command executes only if the wallet is unlocked. If that is feasible, `dcrwallet` can prompt for the passphrase here to ensure the wallet is only unlocked for this single operation. `dcrwallet` signs the requested message(s) with requested key(s) and passes signed data back to the caller.
* `politeiavoter` proceeds with the signed data (e.g. sends it to Politeia server)

In the above example `politeiavoter` is used, but it could be any other program that needs to sign data. Examples:

- command line tool to stake with VSP that implements new accountless API of dcrstakepool (https://github.com/decred/dcrstakepool/issues/574)
- dcrdex client (https://github.com/decred/dcrdex/issues/36)
 
Bonus:

* If this is implemented, it will pave the way to signing Politeia votes with other signing devices like Trezor. It seems reasonable to use `dcrwallet` to talk to Trezor, so that `politeiavoter` does not need to implement Trezor-specific API.
* Offline signing becomes reachable. Signing request and response need to be saved as files and passed between online and offline machines (USB, CD/DVD, QR codes). Since it includes multiple messages at once, only one signing roundtrip is necessary, and then the trickler can slowly send signed votes.
* The benefit for `politeiavoter` is that it would need a bit less paranoid (although still decent) audit because it could do less damage without wallet private passphrase.

Discussions:

* [2019-02-26](https://matrix.to/#/!VFRvyndKpzcLrVslQD:decred.org/$15512027129365SkNlj:decred.org)
* [2019-03-12](https://matrix.to/#/!VFRvyndKpzcLrVslQD:decred.org/$155236122824179HUtQi:decred.org)
