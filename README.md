# Swarm Community Calls

A static site listing past and upcoming [Swarm Community Calls](https://www.ethswarm.org) — monthly online events hosted by Swarm Foundation every last Thursday of the month.

## What's inside

- **Upcoming call banner** — date, agenda, and links to join on X, add to calendar, and submit AMA questions
- **Past call cards** — video thumbnails, key topics, links to watch the recording and read the recap

## Adding a new call

Use the built-in Claude Code skill:

```
/add-call
```

It will guide you through two workflows:

- **Add past call card** — run after the recap blog post is published; inserts the new card and rolls the upcoming banner to the next date
- **Update upcoming agenda** — run ~1 week before the call when the agenda is known; replaces the generic subtitle with the agenda list

## Stack

Plain HTML + CSS. No framework, no build step — open `index.html` in a browser or deploy to any static host.

## Assets

- `assets/` — logos and call visuals
- Blog post thumbnails are loaded directly from `blog.ethswarm.org/uploads/`

## Links

- [Swarm Foundation](https://www.ethswarm.org)
- [Swarm Blog](https://blog.ethswarm.org)
- [Discord](https://discord.gg/9EJTdUKa)
- [X / Twitter](https://x.com/ethswarm)
