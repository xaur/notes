---
published_url: https://github.com/decred/politeiagui/issues/845
---

Politeia UX is very similar to Reddit UX and it fires up my Reddit neurons all the time. The fact that my comments are not automatically upvoted by me is super confusing. Besides confusion it leads to comment up/downvote score distortion as some authors upvote their comments while others don't.

I see two approaches:

1. Upvote own comments by default and allow to un-upvote
   * like Reddit, this reduces confusion by reducing UX difference with Reddit
   * allowing to un-upvote and even downvote own comment later has a use case: if I realize tomorrow that my comment is junk I can remove my upvote and add a downvote
1. Do not upvote own comments by default _and_ do not allow it
   * this is less confusing than current behavior because it leaves no choice
   * this is still more confusing for heavy Reddit users
   * if upvoting own comments is not allowed, so should be downvoting

I suggest implementing first approach. If that happens I also suggest automatically upvoting all comments by on behalf of their authors to remove any score distortion.
