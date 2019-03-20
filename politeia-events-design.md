# Event-based design for feeds and notifications

## Overview

Activity feeds similar to those in Reddit would significantly improve the UX and awareness on Politeia.

This design can support features to help users stay on top of any activity:

* global activity feed (politeiagui #783) - like Reddit `/r/subreddit/comments` page
* personal subscriptions feed (politeiagui #844) - like Reddit `unread` page
* proposal activity/history feed (politeiagui #820)
* user activity feed - like Reddit `/user/name` page
* history of username changes (politeiagui #847)

The design can also be useful for:

* manual read state tracking (politeiagui #1021)
* highlighting of unread comments (politeiagui #732)

Feeds must be served very fast and therefore need to be precomputed and cached. The more immutable is the data, the easier and more robust will be the caching.

The design is partly inspired by Matrix protocol.

## Abstractions

1. `event`
2. `filter`
3. `subscription`
4. `feed`
5. `notification`
6. `template`

### event

`events` capture any activity that happens in the system. Events are immutable and form immutable list.

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
  * verified email (or other contact), paid paywall, warned by mods, blocked|unblocked in proposal X, suspended|resumed globally

Comment upvote and downvote can be alternatively expressed as a standalone type 'comment vote'. {TODO: see if this simplifies the design.}

Events have different structure depending on their types.

Sources (or domains) where events can originate from (in realtime or in playback):

* public proposals Git repo
* private proposals Git repo
* server memory that was not yet flushed to Git
* server memory that is not going to be flushed to Git

### filter

`filter` matches an `event` against some condition and returns true|false.

Example filters:

* user-agnostic: `global feed` and `new comments`
* per-proposal: `activity for proposal X` and `activity for proposal X sans comments`
* per-user: `comments by user X`
* personalized: `all activity user X subscribed for` (complex, composite)

Example filter definitions:

* `global feed`: (NOT (type = comment state change, action = upvote OR downvote))
  * almost everything, exclude some spammy event types
* `new proposals`: (type = new proposal)
* `comments for proposal X`: (type = new comment, proposal id = X)
* `activity for proposal X`: (type = new comment OR proposal status change OR new proposal version, proposal id = X)
* `activity for proposal X sans comments`: (type = proposal status change OR new proposal version, proposal id = X)
* `immediate replies of comment X`: (type = new comment, parent comment id = X)
  * this is a bit more complex (and requires more expressiveness) than filters based on fields of a single event

### subscription

`subscription` tells the server to watch for certain `events`, generate `notifications` and deliver them to the `user`.

Subscriptions are created explicitly or implicitly by the user, or by the server if it sees a need to subscribe the user to something.

Examples of explicit subscriptions:

* subscribe to all `new proposals` action
* subscribe to all `comments for proposal X` action
* subscribe to all `activity of user X` action ("follow X")

Examples of implicit subscriptions:

* subscribe to `immediate replies of comment X` after user posted comment X
* subscribe to all `comments for proposal X` after user posted proposal X

A `subscription` consists of who, where and what (subject and format):

* who: user identifier
  * UUID or pubkey
* where: user `contact` or `destination` for delivery
  types: on-site 'unread' virtual inbox, web socket, email address, Matrix room, Twitter account
* what - subject: a `filter` for events
  * e.g. new proposal, proposal state change, new comment
* what - format: a `template` - restricted by `destination` type

An advanced feature is aggregation of multiple events into a single notification by time interval. This adds a 'when' element to the subscription: a schedule when to send stuff. Options: immediately (no aggregation), every X hours, every X days, weekly. This feature requires to accumulate events until the notification can be executed.

Even more complex aggregation is possible, e.g. "daily activity for proposal X" so if 3 proposals get activity, they will generate 3 notifications.

For extra robustness, the server can record (event id, user id) pairs for events that were successfully delivered to the user through at least one route. If there is a programming error and some events were not generated, or some subscriptions didn't work, events missed by the user could be identified later by checking the re-generated (and fixed) list of events against this events-users table.

### feed

`feed` is an immutable, append-only list of `events` that passed certain `filter`.

Feed can be expressed as a special kind of intra-server `subscription`.

Feeds can serve events rendered in various formats (HTML, JSON, RSS).

Since events are immutable, their rendered versions are immutable too and can be easily cached.

Some trivial events like comment up/down votes do not belong to most feeds, but a filter or feed to show them might be interesting for moderators or analysts.

### notification

`notification` is sent to a user to serve his `subscription`.

Generated from `events` hitting `subscriptions`.

A single event can hit multiple overlapping subscriptions with the same `user` or same `destination`. In case of aggregating subscriptions, multiple events can hit the same aggregation window. Therefore, multiple events and multiple subscriptions can yield a single notification. Some deduplicating mechanism is necessary to avoid notifying the user twice for the same event, or for different events in the same aggregation window.

For extra robustness and ability to replay errored notifications, the server can track their delivered state.

{TODO: how does delivery tracking play with aggregation?}

{TODO: does delivery boolean belong to (subscription, event) pair, notification, or both?}

### template

`template` is a piece of text with gaps that can be filled with variables.

Templates can be

* designed for `notifications`, when rendered they are sent to the user.
* designed for `events`, when rendered they populate `feeds`

Possible formats: HTML snippet, email, RSS, JSON for XHR response, JSON Matrix message, tweet.

In addition to the central object for which the event is crafted (`event` or `notification`), context may be needed to fill some variables in the template (e.g. user name, base server URL, etc). Try to minimize the amount of context needed.

## Processing

Server collects events in memory and processes them against subscriptions. Some subscriptions need to be served faster, while some slower (e.g. daily digest). It is reasonable to process events in batches every X time units, say every 5 minutes.

## Tools

Several tools can be built:

1. a tool that can rebuild the list of events from scratch
   * inputs: proposal Git repository (-ies) and/or server memory
   * can help to debug and recover from failures
2. a tool that can "play back" a time range of `events` against a set of `subscriptions`
3. a tool to re-send any undelivered notifications

