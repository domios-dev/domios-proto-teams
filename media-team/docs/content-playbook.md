# Content playbook

How the media team runs a product's content week by week. The team prepares; the owner publishes and approves anything public.

## The weekly loop

1. **Plan.** The Producer plans the week per channel, with **at most one post per day per channel** (different channels may post on the same day). Times are in the owner's timezone. Every planned post is a catalog line (date, time, channel, campaign, status, code) plus one Markdown file with its text.
2. **Write.** The `writer` drafts from real material: what happened this week, a number, a fix. Each channel gets its own text for the same story. Run the humanizer pass.
3. **Assets.** Covers and clips go to `video_editor` or the design lead as a brief file (message, channel, size, length, date, exact caption text).
4. **Owner review.** Show the exact text and asset. The owner publishes themselves.
5. **Mark published.** Set the status, the real URL and the publish time. Derive the publish time from the post id where the platform encodes it; see `metrics-pipeline.md`.
6. **Measure.** Collectors run on a schedule, and the dashboard updates itself.
7. **Learn.** The `analyst` compares each post with its channel's own baseline (median views, hit line). The Producer rewrites the next week from what worked and drops what did not.

## Content rules

- **Every post rests on something real:** a concrete event, a hard number, or a practical step-by-step fix for a real pain. Before writing, ask: what actually happened, what is the number, what exact steps solve it? If there is no answer, there is no post yet.
- **No generic theses.** In practice they land at about half a channel's median views.
- **Native language on local pains travels far.** A practical guide in the audience's language can beat English opinion posts by an order of magnitude.
- **Never promote evading a platform's rules or restrictions,** even if it would get reach.
- **Never invent facts, numbers, quotes or reactions.**
- **Endings:** stop on the last concrete beat. No labelled takeaways ("My take:") and no closing question to "the audience". Ask a question only rarely, when the answer is genuinely wanted.
- **Captions and claims match what the product actually does,** for example "a couple of clicks", not "one line", if it takes clicks.

## The formula that travels

- **The shape.** Name a specific audience's pain in the first line, then "I did X", then numbered steps someone can repeat, then a reason to follow, such as what the next part covers.
- **Why it works.** Feed rankers reward time spent reading and the actions people take afterwards (profile visits, follows). Steps keep people reading.
- **Watch the profile.** Once a post travels, the leak is usually there, not in the post. If the profile has no bio, no link and nothing pinned, the reader leaves, so fix it before the next post.
- **Professional networks.** What travels is a named, verifiable event or fresh news about a well-known tool, plus your own concrete experience with numbers. Overused theses ("one model is a single point of failure", "agents have amnesia") stall at the median however well they are written.
- **Pick the language per channel by audience,** not by habit, and record it as an owner rule.

## Variants and the owner's pick

- **Variants.** Every planned post gets 5 variants by default, each a different angle, in `<post file stem>.variants/A.md … E.md`. Each file starts with `angle: <≤60 chars>`, then a blank line, then the post text in the usual format. Write more only on request; the catalog accepts keys A–Z.
- **The pick.** The owner compares, copies, chooses or rejects in the catalog's Planned view. Never ask them to pick through notifications.
- **After the pick.** The Producer copies the chosen body into the post file unchanged. Variants of a published post are frozen.
- **Rejections are data.** Read them before writing the next variants. A rewritten rejected variant comes back as a new candidate.

## Links and attribution

- Every link to the product site is a **short tracked link** `https://go.<domain>/<code>`, where `<code>` is the post's permanent catalog code.
- One redirect rule at the DNS or CDN level sends `go.<domain>/*` to `https://<site>/?utm_medium=social&utm_content=<code>`. The site's analytics SDK reads `utm_content`, and the catalog maps the code to the post, channel and campaign.
- **Do not invent other link schemes.**
- On channels where links in the body cut reach (LinkedIn), the link goes **only in the first comment** (the `## Reply` section of the post file).

## Post file format

The text comes first. A line `## Reply` starts the first comment or reply, and `---` lines separate thread parts. The file holds the post text only; metadata lives in the catalog line.

## Owner gates

- Nothing public without the owner's explicit OK for that item: posts, replies, emails, listings and any site change.
- No paid spend unless the owner decides it.
- Agents never handle passwords. The owner signs in. When a login is needed, stop and ask with a choice notification.
- Ask the owner only for what blocks the work, as choice questions, with your recommendation first. Bring results and links, not questions.

## Corrections become rules

When the owner corrects something, turn it into a durable rule the same day: team memory, the project AGENTS.md and the relevant guide or skill. The same correction must never be needed twice.
