---
name: antwork-viral
description: Use when the user wants to learn from posts that went viral in their niche or from a creator they admire, and turn the patterns into posts for their own business through Antwork. Trigger on phrases like "what's going viral about X", "find top posts on #hashtag", "study @creator's best posts", "what works in my niche", "make posts like this one but for us", "steal like an artist", or any request to analyze other people's high-performing posts and write variations. Reads the outliers, names the patterns, plans variations in the user's voice, and drafts only after the plan is approved.
---

# Studying viral posts and writing variations in Antwork

The value here is not "copy what went viral". It is **why** a post beat its author's usual numbers, applied to the user's own business, in the user's own voice. Antwork reads and ranks the posts; you find the pattern and write.

## 1. Pin down the target

You need three things before reading anything. Ask only for what's missing:

- **Platform.** Discovery reads X, YouTube, Bluesky and Mastodon. For anything else (LinkedIn, Instagram, TikTok, Threads, Facebook, Pinterest...) the tool answers `unsupported`: ask the user to paste the posts they like (text, link or a screenshot) and skip to step 3 with those.
- **Where to look.** A niche (`source="search"` with keywords or a `#hashtag`) or a creator (`source="account"` with their `handle`; on Mastodon `user@server`). A niche finds what broke out among many accounts; a creator shows which of their own posts beat their usual.
- **Whose posts we're writing.** The user's account the variations are for. It can be on a different platform from the one you study (study X, write for LinkedIn).

## 2. Read the user's voice, then the outliers

1. `get_post_context(account_id=...)` for the account you'll write for. Its `recentPosts` are the voice; its `performance.topPosts`, when present, are what already works for them. Read these first: the variations have to sound like this account, not like the viral author.
2. `fetch_platform_posts(platform=..., source="search", query=...)` or `fetch_platform_posts(platform=..., source="account", handle=...)`. Optional: `since_days` (1-30; default 7 for search, 30 for a creator; X search reaches 7 days), `limit` (1-25, default 10), `account_id` (whose connected sign-in makes the read, when the workspace has several on that platform).

Read the results like this:

- Posts come back sorted by `outlierScore`: how many times its usual a post did. 1 is typical; 5 means five times. `scoreBasis` says against what: `followers` (engagement per follower across the results), `set` (raw numbers, when follower counts were missing) or `author` (a creator against their own median).
- Each post carries `text` (clipped long), `url`, `author`, `format`, `metrics` and `traits` (length bucket, first line, hashtag count, has a link, has a question). Open the `url` only if you need what the text doesn't show.
- A read is reused for a day, so asking again in the same day costs nothing and returns the same posts.

Refusals and what to say:

| Answer | Meaning | What to do |
|---|---|---|
| `discovery_requires_upgrade` | Free plan | Tell the user it's on paid plans, with the `upgradeUrl` from the answer. Offer the paste route instead. |
| `no_connected_account` | No account on that platform in this workspace | The read uses a connected account's sign-in. Offer `get_connection_urls`, or the paste route. |
| `discovery_x_weekly_limit` | The weekly X search budget is used up | Try YouTube, Bluesky or Mastodon, a creator on X instead of a search if the message allows it, or wait. |
| `creator_not_found` | The handle doesn't exist on that network | Check the spelling with the user; Mastodon needs `user@server`. |
| `reconnect_required` | The connected account's sign-in expired | Its `reauthUrl` from `list_social_accounts`. |
| `platform_rate_limited` / `platform_unavailable` | The network is throttling or down | Say so and try later; don't loop. |
| `unsupported: true` | Platform with no legal way to read other people's posts | The paste route. |

## 3. Name the patterns, not the topics

Pick the 3 to 5 posts with the highest `outlierScore` (skip ones clearly off-topic for the user, and say you did). For each, one line on *why* it worked:

- **Hook**: the first line's mechanism (bold claim, number, contrarian take, question, story opener, "I did X so you don't have to").
- **Format**: thread, list, single image, carousel, short video, link post.
- **Length** and **structure**: one punchy line, or a long post with a payoff at the end.
- **Angle**: what the reader gets (a shortcut, a warning, a surprising number, permission, a behind-the-scenes look).

Then name the **2 or 3 patterns** they share. "Two of the top three open with a number and promise a shortcut, and both are under 200 characters" is useful. "They're about AI" is not: the topic is theirs, the pattern is reusable.

Be honest about thin data: with 3 posts or a `set` basis, say the pattern is a hint, not a law.

## 4. Plan variations on the user's business, then get approval

Turn each pattern into variations about **the user's** product, story or expertise, in **their** voice. Give a numbered plan, one row per post:

- **Pattern** it uses (and which source post shows it).
- **Account** and platform it's for.
- **Hook**: the actual first line, written in the account's voice.
- **Angle**: one sentence on the post.

Rules for every variation:

- **Structure, never wording.** Borrow the shape; write every sentence fresh. No lifted phrases, no paraphrase that tracks the original line by line.
- **Quotes name their creator.** If the user wants to cite or react to a post, credit the author and link it.
- **Fit the target platform.** A pattern from X still has to respect LinkedIn's or Instagram's norms and limits; Instagram, Pinterest, TikTok and YouTube need media.
- No AI tells in hooks (see `antwork-voice`).

Stop here and wait for the user to pick or edit. No `create_post` before the plan is approved.

## 5. Draft and hand off

For the approved rows: one post is `create_post(account_id=..., text=...)`; the same idea on several accounts is `create_post(posts=[{account_id, text}, ...])`, each entry in that account's voice. Then follow the server's next step (publish now, schedule, or keep as drafts).

To spread approved variations over the next weeks, hand off to **`antwork-calendar`**; to build a sequence around one pattern, **`antwork-campaign`**.
