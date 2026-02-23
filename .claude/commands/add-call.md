# Community Call Skill

Three workflows — tell the agent which one you want:

- **"create upcoming"** — right after a call: roll the banner to the next call + create past call card
- **"update agenda"** — ~1 week before: add agenda to the upcoming banner
- **"add recap"** — ~7 days after a call: add recap URL + banner image to the past call card

---

## Workflow A: Create Upcoming

Triggered when: the next call date is known (typically right after the previous call).

### Steps

1. **Collect info from the user:**
   - Next call date (e.g. "March 27, 2026")
   - X broadcast link for the next call (if not yet available, use `#`)

2. **Read the current upcoming banner** in `index.html` to extract the outgoing call's data:
   - Date (from `.call-featured-date`)
   - Watch on X link (from `.call-featured-cta` href)
   - Agenda items (from `.call-featured-agenda` `<li>` elements)
   - Title month (from `.call-featured-title`)

3. **Create a new past call card** using the extracted data. Insert at the TOP of `.call-list` (before the first `<article class="call-card">`):

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
      <!-- TODO: replace placeholder with banner image once recap is published -->
      <a href="[WATCH_ON_X_URL]" class="call-video-thumb" target="_blank" rel="noopener">
        <div class="call-video-placeholder">
          <svg viewBox="0 0 24 24" width="48" height="48"><path fill="currentColor" d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-4.714-6.231-5.401 6.231H2.744l7.73-8.835L1.254 2.25H8.08l4.259 5.631 5.905-5.631zm-1.161 17.52h1.833L7.084 4.126H5.117z"/></svg>
          <span>Watch on X</span>
        </div>
      </a>
    </div>

    <ul class="call-topics">
      [PASTE AGENDA ITEMS AS <li> HERE]
    </ul>

    <div class="call-links">
      <a href="[WATCH_ON_X_URL]" class="call-link" target="_blank" rel="noopener">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="currentColor" d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-4.714-6.231-5.401 6.231H2.744l7.73-8.835L1.254 2.25H8.08l4.259 5.631 5.905-5.631zm-1.161 17.52h1.833L7.084 4.126H5.117z"/></svg>
        Watch on X &rarr;
      </a>
      <!-- TODO: add recap link once published (~7 days after call) -->
    </div>
  </div>
</article>
```

4. **Create a new `.ics` file** at `calendar/swarm-community-call-[month]-[year].ics`:
   - Use the event datetime: new call date at 17:00 CET (16:00 UTC), duration 1 hour (DTEND = DTSTART + 1h)
   - Include the agenda in the description if known, otherwise use generic text
   - Follow the same format as existing `.ics` files in `calendar/`

5. **Update the upcoming banner** in `index.html`:
   - Update `.call-featured-date` with the new date
   - Update `.call-featured-title` with the new month (e.g. `Swarm Community Call &mdash; March`)
   - Update the Watch on X button `href` with the new broadcast link (or `#` if not yet known)
   - Reset `.call-featured-subtitle` back to generic: `This month&rsquo;s agenda:`
   - Replace `.call-featured-agenda` with a single placeholder item: `<li>Agenda coming soon</li>`
   - Update the countdown target date in the `<script>` (the `eventTime` variable)
   - Update all calendar deep link URLs in the `.cal-picker-dropdown` with the new date/title
   - Update the `.cal-picker-dropdown` Apple/iCal `.ics` href to the new file

---

## Workflow B: Update Agenda

Triggered when: ~1 week before the call, the agenda is known.

### Steps

1. **Collect info from the user:** 3–5 agenda items for the call.

2. **Update the `.call-featured-agenda`** in `index.html` — replace existing items with the new agenda:

```html
<li><strong>[Topic heading]</strong> &mdash; [one line summary]</li>
```

---

## Workflow C: Add Image

Triggered when: user provides the call banner image (before the recap is published).

### Steps

1. **Collect info from the user:**
   - Image filename (they will drop it into the `assets/` folder, e.g. `0326.png`)

2. **Update the most recent past call card** (top of `.call-list`):
   - Find the card's `call-video` div containing the `call-video-placeholder`
   - Replace the placeholder block with the thumbnail + play overlay, linking to the Watch on X URL already in the card's `.call-links`:
   ```html
   <a href="[WATCH_ON_X_URL]" class="call-video-thumb" target="_blank" rel="noopener">
     <img src="assets/[IMAGE_FILENAME]" alt="[Month Year] Community Call">
     <div class="call-video-play">
       <svg viewBox="0 0 24 24"><path fill="currentColor" d="M8 5v14l11-7z"/></svg>
     </div>
   </a>
   ```
   - Remove the `<!-- TODO: replace placeholder... -->` comment.

---

## Workflow D: Add Recap

Triggered when: ~7 days after the call, the recap blog post is published.

### Steps

1. **Find the recap URL.** If not provided by the user, derive it from the live sitemap:
   ```
   curl -s "https://blog.ethswarm.org/foundation/sitemap.xml" | grep -o 'https://blog.ethswarm.org/foundation/[^<]*swarm-community-call[^<]*'
   ```
   Match by call date/month.

2. **Update the most recent past call card** (top of `.call-list`):
   - Add the "Read Recap" link inside `.call-links`, after the Watch on X link:
   ```html
   <a href="[RECAP_URL]" class="call-link" target="_blank" rel="noopener">
     <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="currentColor" d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8l-6-6zm-1 2l5 5h-5V4zm-3 10H8v-2h2v2zm4 0h-2v-2h2v2zm0 4H8v-2h6v2z"/></svg>
     Read Recap &rarr;
   </a>
   ```
   - Remove the `<!-- TODO: add recap link... -->` comment.

---

## Notes
- Title format is always: `Swarm Community Call, DD Month` (e.g. `Swarm Community Call, 26 February`)
- For December Solstice specials, use `<span class="call-episode">Solstice Special</span>` instead of "Community Call"
- Agenda topics should be concise: `<strong>Short label</strong> &mdash; one sentence max`
- The file to edit is `/Users/gzupan/Downloads/Repos/swarm-community-calls/index.html`
- Calendar files live in `/Users/gzupan/Downloads/Repos/swarm-community-calls/calendar/`
