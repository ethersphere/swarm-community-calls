# Add Community Call Card

This skill handles two separate workflows. Tell the agent which one you want:

- **"add call"** — after the call happens: add the past call card and roll the upcoming banner to the next date
- **"update agenda"** — one week before the call: add the known agenda to the upcoming banner

---

## Workflow A: Add Past Call Card

Triggered when: the call has happened and the recap blog post is published.

### Steps

1. **Collect info from the user** — ask for:
   - Call date (e.g. "February 26, 2026")
   - X broadcast link (e.g. `https://x.com/i/broadcasts/...`)
   - Recap blog URL (e.g. `https://blog.ethswarm.org/foundation/2026/swarm-community-call-...`)

   If the user doesn't provide the recap URL, derive it from the live sitemap:
   ```
   curl -s "https://blog.ethswarm.org/foundation/sitemap.xml" | grep -o 'https://blog.ethswarm.org/foundation/[^<]*swarm-community-call[^<]*'
   ```
   Match by call date to find the correct slug.

2. **Fetch the recap from the blog repo** to extract:
   - Banner image filename (from `banner = "/uploads/..."` in frontmatter)
   - 3 key topics from the content (pick the 3 most significant agenda items)

   The recap files live at:
   `https://github.com/ethersphere/ethswarm-blog-hugo` → `/content/foundation/posts/swarm-community-call-[month]-[year]-recap.md`

   Use gh CLI:
   ```
   gh api repos/ethersphere/ethswarm-blog-hugo/contents/content/foundation/posts/FILENAME.md --jq '.content' | base64 -d
   ```

3. **Update index.html** — insert the new card at the TOP of the `.call-list` div (before the first `<article class="call-card">`), using this template:

```html
<!-- [Month] [Year] -->
<article class="call-card">
  <div class="call-card-content">
    <div class="call-meta">
      <span class="call-episode">Community Call</span>
      <span class="call-date">[Full date, e.g. February 26, 2026]</span>
    </div>
    <h2 class="call-title">Swarm Community Call, [DD Month]</h2>

    <div class="call-video">
      <a href="[X_BROADCAST_URL]" class="call-video-thumb" target="_blank" rel="noopener">
        <img src="https://blog.ethswarm.org/uploads/[BANNER_FILENAME]" alt="[Month Year] Community Call">
        <div class="call-video-play">
          <svg viewBox="0 0 24 24"><path fill="currentColor" d="M8 5v14l11-7z"/></svg>
        </div>
      </a>
    </div>

    <ul class="call-topics">
      <li><strong>[Topic 1 heading]</strong> &mdash; [one line summary]</li>
      <li><strong>[Topic 2 heading]</strong> &mdash; [one line summary]</li>
      <li><strong>[Topic 3 heading]</strong> &mdash; [one line summary]</li>
    </ul>

    <div class="call-links">
      <a href="[X_BROADCAST_URL]" class="call-link" target="_blank" rel="noopener">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="currentColor" d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-4.714-6.231-5.401 6.231H2.744l7.73-8.835L1.254 2.25H8.08l4.259 5.631 5.905-5.631zm-1.161 17.52h1.833L7.084 4.126H5.117z"/></svg>
        Watch on X &rarr;
      </a>
      <a href="[RECAP_URL]" class="call-link" target="_blank" rel="noopener">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="currentColor" d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8l-6-6zm-1 2l5 5h-5V4zm-3 10H8v-2h2v2zm4 0h-2v-2h2v2zm0 4H8v-2h6v2z"/></svg>
        Read Recap &rarr;
      </a>
    </div>
  </div>
</article>
```

4. **Roll the upcoming call banner** — update the featured section at the top of the page:
   - Update the date to the next call (next last Thursday of the month)
   - Reset the subtitle back to the generic text: `Join the next monthly community call. Get the latest Swarm development update, ask questions live, and connect with the ecosystem.`
   - Clear the X broadcast link to `href="#"` until the new broadcast URL is available
   - Update the Add to Calendar link if a new addevent URL is provided, otherwise set to `href="#"`

---

## Workflow B: Update Upcoming Banner with Agenda

Triggered when: ~1 week before the call, the agenda is known.

### Steps

1. **Collect info from the user** — ask for:
   - 3–5 agenda items for the call (topics that will be covered)

2. **Update the `.call-featured-subtitle`** in index.html — replace the generic subtitle with a concise agenda list using this template:

```html
<p class="call-featured-subtitle">This month's agenda:</p>
<ul class="call-featured-agenda">
  <li>[Agenda item 1]</li>
  <li>[Agenda item 2]</li>
  <li>[Agenda item 3]</li>
</ul>
```

   Add to styles.css if `.call-featured-agenda` doesn't exist yet:
```css
.call-featured-agenda {
  list-style: none;
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 0.9375rem;
  color: var(--color-text-secondary);
  max-width: 520px;
}

.call-featured-agenda li {
  padding-left: 14px;
  position: relative;
}

.call-featured-agenda li::before {
  content: '';
  position: absolute;
  left: 0;
  top: 10px;
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: var(--color-orange);
}
```

---

## Notes
- Topics/agenda items should be concise: one short phrase per item, no full sentences needed
- Title format is always: `Swarm Community Call, DD Month` (e.g. `Swarm Community Call, 26 February`)
- For December Solstice specials, use `<span class="call-episode">Solstice Special</span>` instead of "Community Call"
- The file to edit is `/Users/gzupan/Downloads/Repos/swarm-community-calls/index.html`
