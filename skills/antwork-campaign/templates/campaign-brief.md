# Campaign Brief — {{campaign name}}

## Goal
{{The single outcome this campaign drives — e.g. "300 waitlist signups by launch day."}}

## Success metric
{{One measurable number, and how it'll be read from get_performance.}}

## Audience
{{Who this is for — the reader you're writing every post toward.}}

## Key message
{{The one through-line every post reinforces, in one sentence.}}

## Platforms / accounts
| Account / Platform | `account_id` | Voice fresh? | Role |
|--------------------|--------------|--------------|------|
| @handle / linkedin | {{id}} | {{yes / stale → refresh}} | {{primary / amplify}} |
| @handle / x | {{id}} | {{...}} | {{...}} |

## Cadence
{{How many posts, which days, over what window. Reference templates/launch-week.md for launch arcs.}}

## CTA
{{The action each post asks for — keep it consistent across the campaign.}}

## Post map
| # | Day | Account | Role in arc | Hook | Per-post `goal` |
|---|-----|---------|-------------|------|-----------------|
| 1 | {{Mon}} | {{@handle/x}} | tease | {{hook}} | tease |
| 2 | {{Wed}} | {{@handle/linkedin}} | reveal | {{hook}} | launch |
| 3 | {{Fri}} | {{@handle/x}} | proof | {{hook}} | social-proof |
| 4 | {{...}} | {{...}} | last-call | {{hook}} | last-call |

---
Set a meaningful per-post `goal` so analytics stays legible. One row = one
`create_post` (`posts=[...]` when the row runs on several accounts) → `schedule_post`.
