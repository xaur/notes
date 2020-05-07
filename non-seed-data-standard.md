---
title: Standard format for aux data export
published_url: https://github.com/decred/dcrwallet/issues/1731
---

Decred wallets use some data that cannot be generated from the seed:

- account names
- account purpose/behavior
- redeem scripts for the VSPs
- arbitrary JSON configuration for VSPs

It would be nice to:

1. Preserve all this data in a standard format respected by all software
2. Export from/import into the wallet in one action
   - including an option to import when creating a new wallet or restoring from seed
3. Encrypt the bundle since it contains sensitive data

Having that would improve the UX by avoiding the boring steps of (re)configuring new and restored wallets. And perhaps even eliminate some user errors where they forget to save/restore their redeem scripts.

So the idea is to define a standard format for non-seed wallet data that is respected by dcrwallet, Decrediton and the mobile apps. Then, add commands to export and import a bundle of such data to all participating wallets.

The purpose/behavior account tags address the following [point](https://matrix.to/#/!HEeJkbPRpAqgAwhXWO:decred.org/$1588686448154Uglpx:zettaport.com) by @jrick:

> looking a bit longer term, i would like if accounts did know what they were being used for and enable or change behavior based on that  
> there is a slight problem during seed restores though, since those prefs/configs can't be restored off the chain

In the future, it may be extended to include an addressbook (name->addr map) and other stuff.

The bundle file should be encrypted with a passphrase by default, but with a way to opt-out and save/load an uncencrypted file.

Possible data structure is presented below. Note that multiple json files like this can store all your wallet setups (cold, desktop, mobile, voting, etc):

```json
{
    "schema": "decred_wallet:v1",
    "name": "main",
    "description": "My main desktop wallet",
    "accounts": [
        {
            "index": 0,
            "name": "default",
            "type": "regular",
        },
        {
            "index": 1,
            "name": "unmixed",
            "type": "cspp_unmixed",
            "cspp_group": "main",
            "keep_unlocked": true,
        },
        {
            "index": 2,
            "name": "mixed",
            "type": "cspp_mixed",
            "cspp_group": "main",
        },
        {
            "index": 3,
            "name": "voting1",
            "type": "voting",
            "note": "My note about how I use this",
        },
        {
            "index": 4,
            "name": "tips",
            "type": "regular",
        },
    ],
    "vsps": [
        {
            "code": "Lima",
            "homepage": "https://ultrapool.eu",
            "api_base_url": "...",
            "api_token": "abc...",
            "redeem_script": "abc...",
        },
    ],
    "extensions": [
        {
            "schema": "dcrwallet:v1",
            "spv_enabled": false,
            "spv_use_ugly_ca_certs": false,
            "gap_limit": 20,
            "cspp_enabled": true,
            "cspp_footshot_protection_enabled": true,
            "ticketbuyer_enabled": false,
            "cache_passphrase_enabled": false,
        },
        {
            "schema": "decrediton:v1",
            "fancy_animations_enabled": false,
            "fancy_shadows_enabled": false,
            "gpu_rendering_enabled": false,
            "update_check_enabled": false,
            "fingerprinting_leaks_master_knob_enabled": false,
            "ln_enabled": true,
            "cspp_enabled": true,
            "ticket_based_voting_enabled": true,
            "third_party_requests_enabled": false,
        },
    ],
}
```

When creating a new wallet or restoring from seed in participating wallets, they could have a step that allows to import such file before seed entry.
