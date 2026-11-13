---
name: antwork-analytics
description: Use when the user wants to understand how their social content is performing through Antwork — pulling engagement metrics, ranking top posts, spotting per-platform trends, comparing time periods, or turning numbers into a what-to-do-next report. Trigger on phrases like "how are my posts doing", "what's my best post", "engagement report", "analytics", "which platform is working", "show my growth", "am I gaining followers", or any request for social performance data.
---

# Reporting on performance in Antwork

Antwork's analytics come as data, not charts: `get_performance` returns per-post rows, the per-day and follower numbers as series (columns aligned with a `dates` array, ready to chart), and `findings` already computed from them. Your job is to pull the right numbers, then synthesize them into a decision: what's working, on which platform, and what to make more of. Numbers without a recommendation are not a report.

## 1. Pick the right query — don't pull everything

| Question | Tool | Notes |
|---|---|---|
| "What are my best posts?" | `get_performance(limit, days?, platform?, account_id?)` | `rows`: lifetime totals per published post, plus `platformTotals` and `findings.ranking`. Best for **rankings**. `limit` 1–50 (default 30), `days` 1–365 (default 30). A row's `text` is its first 100 characters (`textTruncated`); `get_post` has the whole post. |
| "How is engagement trending?" / "chart it" | `get_performance(days)` → `daily` | `dates` and `series[platform][metric]`, one array per metric aligned with `dates`: each day's growth (`interval: "day"`), or each week's past 31 days (`interval: "week"`, `dates` are Mondays, the first and last weeks partial). Plus `changeTotals` (the window's growth summed, not lifetime), `reachBasis` per platform and `notes` for days read degraded. Best for **trends over time** and charts. |
| "Am I gaining followers?" | `get_performance(days)` → `followers` | The same `dates`, and per account a `followers` array (weekly: the week's last count), with `approximate` dates and an `unavailableReason` when the count is missing. `findings.followers` has the change over the window. `list_social_accounts` gives each account's latest `followers` (`count`, `asOf`, 30-day `change` since `changeSince`). |
| "How did *this* post grow?" | `get_performance(post_id=..., days?)` | Per-day rows for one post, `cumulative*` and `daily*` columns. Best for **single-post lifecycle**. Omit `days` for full history. |
| "Put it in a sheet" | `get_performance(format="table")` | Flat rows per post and per account, nothing else. `format="rows"` returns `daily` and `followers` as long `{schema, rows}` tables (one row per date and platform or account) and full post rows; it is several times larger, so use it only when asked for daily rows over a long window. |
| "Why is this one post's number blank?" | `get_post(post_id)` | A published post's `metricCells`: every metric with its `state` (see section 3). |
| "When should I post?" | `get_post_context()` | Its `workspace` block carries the configured `preferredTimes` + `timezone`. |

Start with `get_performance` for almost any "how am I doing" ask, then drill in with the others. When the question is what to write next rather than how it went, `get_post_context`'s `performance` block already ranks an account's Antwork-published posts by the metric that matters on its platform (`rankedBy`, `topPosts`, over 90 days) and carries its `followers`.

## 2. Numbers are a few hours old, by design

Metrics refresh on their own every 6 hours, and there is no tool to force a refresh. A post published minutes ago shows its first numbers after the next refresh; say so rather than reporting zeros as a result. Some posts refresh less often: X posts on a free plan are read at 1, 3 and 7 days old only, and metrics stop refreshing once a post is about 30 days old, so older posts carry their final numbers.

## 3. Know which metrics exist, and what a blank means

Beyond likes, comments, shares and `impressions`, the tables carry `reach`, `saves`, `clicks` (any click), `bookmarks`, `linkClicks` (through to the post's URL), `profileVisits`, `follows` (followers gained from the post), `watchTimeSeconds`, and in `get_performance` `avgWatchTimeSeconds`. Each platform reports a different subset. Rules that keep a report honest:

- **A blank (null) is "not reported", never zero.** In `get_performance` a null in a series is a day (or week) not measured: draw a gap, don't drop the line to 0. A metric or platform missing from `series`, or a field missing from a post row, was not measured in the window. A 0 is a measured zero. In `get_post`'s `metricCells` each metric has a `state`: only `measured` is a real number. `unsupported` (with a `reason`: `not_exposed_by_api`, `requires_elevated_scope`, `not_available_for_type`) means the platform cannot give it, `never_fetched` means no successful read yet, `awaiting_permission` means the platform has not released it to Antwork yet, and `unreadable` is an earlier value carried forward after a failed read. Leave all of those out of totals and averages, and write "not reported" in a table rather than 0.
- **`awaitingPermission` and `degraded` columns name the affected fields per row.** `awaitingPermission` is mostly Instagram and Threads (views, reach, saves, shares) while Meta reviews Antwork's insights permission; say the numbers are pending, not that the posts flopped. `degraded` marks values carried forward from a failed read: show them, flagged, but don't build a trend on them.
- **Judge each platform by what it can report.** TikTok video metrics need the `video.list` scope, which is in TikTok's review, so an account without it has TikTok cells that read `unsupported` (`requires_elevated_scope`). X impressions, link clicks and profile visits need elevated API access. YouTube watch time, shares and subscribers gained need the channel's YouTube Analytics grant. Pinterest has no likes, comments or shares; Facebook has no shares. A platform missing a metric is not losing on it: rank and judge it on the metrics it has, and say which ones it lacks.
- **`impressions` means a different thing per platform**, named by each row's `reachBasis` (`impressions`, `views`, `reach`, `reach_30d`). Rows with different bases don't add or compare, and `reach` never adds across posts or platforms. Compare within one platform, or on counters that mean the same everywhere (likes, comments, shares, link clicks, follows).

## 4. Read the table, then interpret

The rows are raw. Turn them into findings:

- **Top performers** — rank within each platform by the metric that matters there, not one number across all of them: saves and shares on Instagram, link clicks on Pinterest, watch time on YouTube, views on TikTok, `totalEngagement` (likes + comments + shares) and link clicks on LinkedIn, X, Facebook and Threads. Use the next one where the first isn't reported. Then look *past* the number to the pattern. What do the top 3 share — format, hook style, topic, length, posting time?
- **Actions beyond likes** — `linkClicks`, `profileVisits` and `follows` show whether a post moved people, not just pleased them. Where they are reported, name the posts that drove them.
- **Follower growth** — from `followers`: the change over the window per account (`findings.followers`), and the days with the biggest rise in the `followers` array (line them up with what was published then; `findings.growth.days` does this already). A missing count comes with an `unavailableReason`; YouTube counts are `approximate` (rounded). Say "not reported" for an account with no counts rather than "no growth".
- **Per-platform trends** — from `daily`: `changeTotals` per platform and the shape of each `series`: which platform is growing, which is flat, where's the effort/return mismatch. A total across platforms adds the platforms' series, except `impressions` with different `reachBasis` and `reach`, which never add.
- **Outliers** — one post 5× the median is a signal, not noise. Name what made it land.
- **Gaps** — platforms with no recent posts, or with traffic but no posting cadence.

## 5. Tie every finding to an action

The report's value is the "so what." Each finding gets a next step, and the strongest ones feed straight into other skills:

- Winning hooks/topics → hand to **`antwork-ideas`** to generate more in the same vein.
- Best posting windows → feed **`antwork-calendar`** when scheduling.
- A high performer worth a sequel → **`antwork-repurpose`** it to other platforms.

## 6. Output

For a quick ask, give a tight terminal summary (top 3 posts + the one thing to do more of). For a full report, write a markdown file following `templates/analytics-report.md` — header with workspace/date/range, executive summary, top performers per platform, follower growth, per-platform trends, and a prioritized recommendations list. Always state the date range the numbers cover, so a stale report can't be mistaken for current.
