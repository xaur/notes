# Event-based design for feeds and notifications

## Overview

Reddit-like activity feeds would significantly improve the UX of Politeia. Features like global activity feed (like Reddit `/r/subreddit/comments` page), personalized subscriptions feed (like Reddit `unread` page), user activity page (like Reddit `/user/name/overview` page) and proposal activity feed, would help users stay on top of all activity that is of interest to them.

These feeds must be served very fast and therefore need to be precomputed and cached. The more immutable is the data, the easier and more robust will be caching.

## Abstractions

Abstractions are marked `as code`.

1. `event`
   * events capture any activity that happens in Politeia
   * events are immutable and form immutable lists
1. `filter`
   * input: an `event`
   * output: true|false
   * example filters:
     * user-agnostic: `global feed` and `new comments` (politeiagui#783)
     * per-proposal: `activity for proposal X` and `activity for proposal X sans comments` (politeiagui#830)
     * per-user: `comments by user X`, `proposals by user X`
     * personalized: `all activity user X subscribed for` (complex, composite)
   * {is filter and query same thing?}
1. `renderer`
   * inputs: `event`, `template`, `context`
   * output: render of the event in target format
   * possible templates: HTML snippet, email, RSS, JSON for XHR response, JSON Matrix message, tweet
   * context may be needed to fill some variables in the template, try to minimize the amount of context needed
1. `template`
   * can be for events (to support feeds)
   * can be for notifications (to support subscriptions)
1. `subscription`
   * consists of `user id`, `destination`, `filter` and `template`
   * user can have multiple subscriptions
1. `notification`

### event

Types:

* new proposal
* new proposal version
* proposal status change
  * submitted, published, abandoned, authorized for vote, vote started, vote finished
* new comment
* comment state change
  * upvote, downvote, censored, flagged, edited (if implemented)
* new user
* user state change
  * verified contact, paid paywall, warned by mods, blocked|unblocked in proposal X, suspended|resumed globally

Comment upvote and downvote can be alternatively expressed as a separate type 'comment vote'.

Events have different structure depending on their types.

Sources (or domains) where events can originate from (in realtime or in playback):

* public proposals Git repo
* private proposals Git repo
* server memory that was not yet flushed to Git
* server memory that is not going to be flushed to Git

### filter

Example filter definitions:

* `global feed`: (NOT (type = comment state change, action = upvote|downvote))
  * almost all, excluding some spammy event types
* `new proposals`: (type = new proposal)
* `comments for proposal X`: (type = new comment, proposal id = X)
* `activity for proposal X`: (type = new comment OR proposal status change OR new proposal version, proposal id = X)
  * alternatively, can be reformulated as multiple OR-ed filters to better fit type system
* `activity for proposal X sans comments`: (type = proposal status change OR new proposal version, proposal id = X)
* `immediate replies of comment X`: (type = new comment, parent comment id = X)
  * this is a bit more complex (and requires more expressiveness) than filters based on fields of a single event

### subscription

`subscription` tells the server to watch for certain `events`, generate `notifications` and deliver them to the `user`.

Subscriptions are created explicitly or implicitly by the user, or by the server if it sees a need to subscribe the user to something.

Examples of explicit subscriptions:

* `subscribe to all new proposals` action
* `subscribe to all comments for proposal X` action
* `subscribe to all activity of user X` action ("follow X")

Examples of implicit subscriptions:

* `subscribe to immediate replies of comment X` after user posted comment X
* `subscribe to all comments for proposal X` after user posted proposal X

A `subscription` consists of who, where and what (subject and format):

* who: user identifier
  * UUID or pubkey
* where: user `contact` or `destination` for delivery
  types: virtual on-site 'unread' inbox, web socket, email address, Matrix room, Twitter account
* what - subject: a `filter` for events
  * e.g. new proposal, proposal state change, new comment
* what - format: a `template` - restricted by `destination` type

An advanced feature is aggregation of multiple events into a single notification by time interval. This adds a 'when' element to the subscription: a schedule when to send stuff. Options: immediately (no aggregation), every X hours, every X days, weekly. This feature requires to accumulate events until the notification can be executed.

Even more complex aggregation is possible, e.g. "daily activity for proposal X" so if 3 proposals are active they will yield 3 notifications.

For extra robustness, the server can record (event id, user id) pairs for events that were successfully delivered through any route. If there is a programming error and some events were not generated, or some subscriptions didn't work, missed events could be recovered later by re-generating a list of events and checking it against this events-users table.

### feed

`feed` is an immutable, append-only list of `events` that passed certain `filter`.

Feed can be expressed as a special kind of intra-server `subscription`.

Feeds can serve events rendered in various formats (HTML, JSON, RSS).

Since events are immutable, their rendered versions are immutable too and can be easily cached.

Some trivial events like comment up/down votes do not belong to most feeds, but a filter or feed to show them might be interesting for moderators or for analysts.

### notification

`notification` is what is sent to a user to serve his `subscription`.

Generated from `events` hitting `subscriptions`.

A single event can hit multiple overlapping subscriptions with the same `user` or same `destination`. In case of aggregating subscriptions, multiple events can hit the same aggregation window. Therefore, multiple events and multiple subscriptions can yield a single notification. Some deduplicating mechanism is necessary to avoid notifying the user twice for the same event, or for different events in the same aggregation window.

For extra robustness and ability to replay errored notifications, the server can track their delivered state.

{TODO: how does delivery tracking play with aggregation?}

{TODO: does delivery boolean belong to (subscription, event) pair, notification, or both?}

## Processing

Server collects events in memory and processes them against subscriptions. Some subscriptions need to be served faster, while some slower (e.g. daily digest). It is reasonable to process events in batches every X time units, say every 5 minutes.

Server iterates through a batch of events and tests them against feeds it must publish to and subscri

## Tools

1. A tool that can rebuild the list of events from scratch
   * inputs: proposal Git repository (-ies) and/or server memory
   * can help to debug and recover from failures
1. A tool that can "play back" a time range of `events` against a set of `subscriptions`
1. A tool to re-send any undelivered notifications

## Applications

* generate various activity feeds (politeiagui #783 #820 #844, user page)
* highlight unread comments (#732)

Inspired by Matrix protocol.
