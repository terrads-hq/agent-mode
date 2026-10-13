---
name: terrads
description: "Create ad creatives for a brand's products with Terrads — list brands, products, angles, audiences and models, create an image or video run, answer its questions, approve its copy, plan or stills and collect the finished images and videos — and check, answer, cancel or finish runs made earlier. Use when the user asks to make, check or fetch Terrads ads or runs."
license: MIT
metadata:
  last-updated: "2026-10-13"
---

# Terrads

Terrads makes finished ad creatives for a brand's products. This skill drives it through the Terrads MCP server (`https://terrads.com/api/mcp`) from your coding agent, such as Claude Code or Cursor, or any MCP client that can send a header. Every tool named below is a tool on the `terrads` MCP server; if that server is not connected, follow Setup.

## What this does to your account

- A run spends the team's credits. The price is only known when the call returns: `creditCost` in the `create_run` reply is what was charged.
- Before `create_run`, estimate the price and agree a ceiling with the user, unless they told you not to ask. `list_models` reports each model's `creditCost`: the credits one render of that model costs today. A run renders every concept in every format, so `creditCost` × `concepts` × the number of `formatIds` is the render part of the price (more again when `references` are attached); the total is higher (a per-run fee, and more for each language other than English). Take that estimate, add headroom, and pass it as `maxCredits`, a whole number of credits.
- If the price is above `maxCredits`, the call answers `quote_changed` with `details.creditCost`, the exact price; nothing is charged and no run exists. Tell the user the price and ask. If they agree, call `create_run` again with `maxCredits` set to that price and a fresh `idempotencyKey`.
- Without `maxCredits` the run is created at the current price, whatever it is. Only omit it when the user said so.
- A video run is the exception: creating one is free (`creditCost` 0, and a number in `maxCredits` is refused). It is charged when you approve its plan with `approve_run_plan`, whose `maxCredits` is required — read the price from `get_run` first and agree it with the user. Re-rolling a still with `reroll_run_still` costs that still's own price.
- Results land in the team's library at terrads.com, next to the runs made in the app.
- A key acts for its whole team, not just the person who created it. Anything you create, cancel or approve is visible to the team.

## Setup

- Terrads API access is included with the Max plan. Keys are created in Settings → API keys at terrads.com and shown once.
- If no `terrads` MCP server is connected in this session, tell the user how to connect one and stop:
  - In Claude Code, the plugin is the recommended path: `/plugin marketplace add terrads-hq/agent-mode`, then `/plugin install terrads@terrads`. Claude Code asks for the key when the plugin is enabled.
  - For any other MCP client, the client setup is documented at https://terrads.com/docs/mcp.
- Never look for a Terrads key in environment files, keychains, shell history or dotfiles, and never ask the user to paste one into the conversation. Ask them to connect the server instead.

## Tools

| Tool | What it does |
|---|---|
| `list_brands` | The team's brands. Every run belongs to one brand. |
| `list_products` | The products the team can advertise, optionally only one brand's, plus the shared system products. |
| `list_angles` | A brand's saved marketing angles: the message an ad leads with. A run needs one. |
| `list_audiences` | A brand's saved audiences. A run takes one, or none. |
| `list_models` | The image models a run may use, each with its current `creditCost` per render. |
| `create_run` | Creates a run for one product, angle and audience across the requested formats and languages. Spends credits; returns a `runId` at once. |
| `get_run` | A run's status, its `kind` (`image` or `video`), its event log, `pendingInput` (the gate waiting on you, or null) and `webUrl` (a video run's page in the app, or null). |
| `list_runs` | The team's runs, newest first, with a cursor for the next page. |
| `answer_run` | Answers every open question of a run paused at the questions gate. |
| `approve_run_copy` | Approves, edits or rejects the copy drafts of a run paused at the copy gate. |
| `cancel_run` | Cancels a run that has not finished yet. Work not yet started is refunded; renders already in progress are not. Cannot be undone. |
| `get_run_results` | The finished creatives as image URLs (a video's MP4 and its poster) with their format and size. The URLs expire. |
| `get_video_options` | What a video brief can name: voices, music, captions, lengths, sizes, styles, and what each AI motion choice makes. |
| `approve_run_plan` | Approves a video run's plans — the charge of a video run — with the AI motion per variation and a credit ceiling. |
| `update_run_plan` | Changes the words of a video plan's scenes (on screen and voice) before approval. Free. |
| `retry_run_plan` | Plans again a video variation that could not be planned. Free. |
| `list_run_stills` | A video run's stills, the pictures its AI clips are made from, with their state and re-roll price. The URLs expire. |
| `reroll_run_still` | Makes one still again, at its own price. |
| `approve_run_stills` | Approves a video run's stills; its AI clips are made next. Free. |

## The workflow

1. Find the inputs. `list_brands`, then `list_products` for the brand; `list_angles` (a run needs an angle) and `list_audiences` (optional); then `list_models`. Every id comes from these calls in this session — never invent one or reuse one from memory.
2. Confirm the brief with the user: product, angle, audience, formats, languages, number of concepts, model, and the credit ceiling from the section above. Then call `create_run` with a new `idempotencyKey` (a short unique string of letters, digits and `_ . : -`) and `maxCredits`. For a video run, set `config.kind` to `"video"` and pick its voice, music, length, style and sizes from `get_video_options` (or leave them to the defaults); the product must be one of the team's own.
3. Poll `get_run` every few seconds while the status is `queued`, `analyzing`, `generating` or `verifying`, passing `after` with the last `seq` you have. When the status is `awaiting_input`, read `pendingInput.gate`:
   - `"questions"`: show the user each question with its options and call `answer_run` with an answer for every open question. A value is an option id from the question, or `custom:` followed by your own short text, or `asset:` followed by the id of an asset the user gives you, where the question allows an image.
   - `"copy"`: show the drafts to the user, unless they delegated copy approval, and call `approve_run_copy` with `drafts` carrying every draft, edited or not, keyed (as strings) by each draft's position in `pendingInput.drafts`, and `rejectedConcepts` listing those same positions for the ones to drop. Those positions are base-creative indexes, not concept numbers: a run makes one base creative per reference per concept, `max(1, references) × concepts` in all, so on a run with two references and three concepts there are six drafts and position 1 is the second of them, not the second concept. A draft that carries a `slots` array instead of the three fields is returned with `slots`, edited or not. Pass `basedOnSeq` from the same `pendingInput`: if the user rewrote a draft in the app after you read it, the call fails with `copy_changed` and its details carry the current drafts — show those and ask again rather than approving stale copy.
   - `"plan"` (a run whose `kind` is `"video"`): show the user each variation's scenes, its Checked lines and the price, and agree the AI motion and the ceiling. `pendingInput.prices.base` is every variation without AI motion; each variation's `motion` gives, per choice, its `credits` with that choice (add `credits` of the pick minus `credits` of none), or `unavailable` when it cannot be picked. Change words with `update_run_plan` (it answers the `basedOnSeq` to use next), re-plan a variation whose status is `"failed"` with `retry_run_plan`, then call `approve_run_plan` with `basedOnSeq`, `motion` and `maxCredits`. That call is the charge; calling it again never charges twice. Pictures, scene lengths and order are changed in the app, at `webUrl`.
   - `"stills"` (a video run with AI motion and the stills check on): `list_run_stills` shows each still; let the user look at them, `reroll_run_still` one they reject (paid; pass its `attempt` as `expectedAttempt`), and call `approve_run_stills` once `pendingInput.ready` equals `pendingInput.stills`.
   - If the user would rather answer a gate in the app, hand it to them: stop polling, give them `webUrl` (a video run's page in the app; null for an image run) and wait for them to tell you it is done.
4. When the status is `completed` (or `failed` or `canceled`, which keep the creatives already finished), call `get_run_results` and hand the user the image or video URLs. Tell them the URLs expire after `expiresInSeconds` and that calling again gives fresh ones; download promptly if the files must be kept.
5. `list_runs` finds earlier runs. `cancel_run` stops one, and only when the user asked to stop it: it cannot be undone, and only work not yet started is refunded.

## Idempotency and duplicates

- `idempotencyKey` names this run. If a `create_run` call fails, times out, or you never see a reply, retry with the same key and the same arguments: a run may already exist and be charged, and the same key replays it (`replayed: true`) instead of charging again. Use a new key only for a genuinely new run, or after `quote_changed`, `too_many_active_runs`, `idempotency_key_spent`, `idempotency_key_reused` or `create_failed`.
- On `duplicate_run`, a run with the same brief was created recently and `details.existingRunId` names it. Do not retry on your own: tell the user, and only if they want a second run call again with `allowDuplicate: true`. Never set `allowDuplicate` pre-emptively.

## Errors

A tool that cannot do what was asked returns an error result carrying the same envelope as the HTTP API, `{ "error": { "code": "…", "message": "…", "details": { } } }`. Branch on `code`. The ones to act on:

- `insufficient_credits` — the run costs more than the team's balance; nothing was charged. Tell the user to top up or shrink the run.
- `quote_changed` — the price is above `maxCredits`; `details.creditCost` is the price and nothing was charged. Ask the user before calling again.
- `rate_limited` — too many calls; wait, then retry with backoff.
- `upgrade_required` — the plan does not include API access.
- `no_subscription` — the team has no active plan.
- `past_due` — the last payment failed; keys resume once the card is updated in the app.
- `duplicate_run` — see the section above.

Errors about the request itself — a revoked key, a plan without access, a spent rate-limit budget — come back on the HTTP response before any tool runs. The full list of codes is the reference page at https://terrads.com/docs/errors.

## Updating

Tell the user they can update with `/plugin marketplace update terrads` then `/plugin update terrads@terrads`, or `npx skills update` where the skill was installed with `npx skills add`.
