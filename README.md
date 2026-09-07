# hamzadev — Hamza Haroon's portfolio

Single-file site. No build step, no dependencies to install: `index.html` carries
all the markup, CSS and JavaScript, and the images sit beside it.

- **Live:** https://hamzadev-eta.vercel.app
- **Repo:** https://github.com/hamzaharoonk07-cloud/hamzadev (branch `main`, which is also what deploys)
- **Weight:** ~200 KB for everything the page loads

---

## 1. Running it locally

```bash
cd hamzadev-main
python -m http.server 8899
```

Then open http://localhost:8899. Opening `index.html` directly with `file://`
mostly works, but the weather fetch is blocked, so use the server.

---

## 2. What is on the page

The current design is **cinematic**: a black, film-styled site with an opening
title sequence, built in the language of the `cinematic-portofilo` project
(Anton display over Oswald, `#de1b1c` red, numbered scenes, grain and vignette).

| Order | Section | id | What it holds |
|---|---|---|---|
| — | Boot screen | `boot` | Progress bar, then an **Enter** button |
| — | Header | — | Wordmark, roles, nav, `Portfolio — MMXXVI` |
| — | Hero stage | `top` | Wordmark, your photo, welcome strip, chips, arrows, dot grid |
| 01 | Who | `about` | The four-paragraph bio |
| 02 | Tools | `stack` | Front end / back end / design / workflow |
| 03 | Work | `work` | The five projects, as a sideways deck |
| 04 | Journey | `journey` | Education timeline |
| 05 | Hire | `hire` | The three services |
| — | Finale | `fin` | Quote, email, social bar, live Karachi clock |

### The opening title sequence

Timed to the reference's own `timeline.js`. The beats **overlap on purpose** —
the letters are still resolving when the welcome strip drops. That overlap is
what separates a title sequence from a queue of fades.

```
0.30  the figure condenses out of the dark, settles into scale
2.10  the wordmark materialises, centre letters first, so the type
      grows around the figure (0.145s stagger, per line)
2.35  one soft flash as it lands — light, not a strobe
2.90  the welcome strip drops from above and overshoots
3.18  chips arrive from the left, 3.32 from the right
3.46  arrows tick in, 3.62 the corner dot grid
4.45  the header fades up and draws its rule across
6.25  settled — ambient breathing and pointer parallax take over
```

The sequence starts **when the visitor enters**, never behind the boot screen.

---

## 3. The five projects

All five are live. Two have public repos.

| # | Project | Live | Repo | Stack |
|---|---|---|---|---|
| 01 | PathSeeker — Career Passport | careerpassporttw.vercel.app | `hamzaharoonk07-cloud/careerpassport` | React, Node, Express, MongoDB, hand-written CSS |
| 02 | CARE Group — Appointment Portal | care-pk.infinityfreeapp.com | `hamzaharoonk07-cloud/care` | PHP 8, MySQL, PDO, Apache, Perl |
| 03 | ShopXtra | shopxtra.store | — | HTML, CSS, JavaScript, Bootstrap, GSAP |
| 04 | PhysioSync | physiosync-pk.vercel.app | — | HTML, CSS, JavaScript, GSAP, Sketchfab 3D |
| 05 | Zaika-e-Karachi | zaikaekarachi.vercel.app | — | HTML5, CSS3, JavaScript, GSAP |

**Every stack above was verified against the deployed site**, not copied from
old notes. The earlier descriptions were wrong in several places: ShopXtra was
listed as a cosmetics demo when it is a working organic grocery on its own
domain; MediCare was a generic "healthcare site" when it is Medicare Hospital,
Rahim Yar Khan; and MediCare and Zaika were both credited with Bootstrap and
jQuery they do not load.

Two notes worth keeping:

- **CARE is on InfinityFree, which serves a JavaScript cookie challenge.**
  `curl` gets nothing back and returns `000`; a real browser loads it fine.
  Verify it with a browser, never a status code. The same challenge means
  link previews in WhatsApp, LinkedIn and Slack will likely come up blank.
- **PhysioSync gates every route behind a profile-creation modal** for new
  visitors, so no automated screenshot can reach the actual app.

---

## 4. Files

| File | Purpose |
|---|---|
| `index.html` | The whole site |
| `profile.jpg` | Your photo — hero figure, and the og:image |
| `logo-*.jpg` | The five project logos, on the cards |
| `shot-*.jpg` | Real screenshots of the five live sites — **not currently used**, kept so they need not be re-captured |
| `resume.pdf` | Linked from the header and the finale |
| `google1fe...html` | Google Search Console verification |

Images were all resized and re-encoded: the page was **2588 KB** and is now
about **200 KB**. `profile.png` was 800×800 at 914 KB for a 260px slot, and
`logo-shopxtra.png` was 2000×2000 at 1310 KB for a 118px thumbnail.

---

## 5. Rules this codebase learned the hard way

Four bugs cost real time here. Each rule exists because breaking it shipped a
broken page.

**1. Never let visibility depend on an animation completing.**
Hidden start states are applied *by script* (`html.cine`), so if the script
fails the page renders plainly instead of blank. On top of that, a check at
1.4s settles everything immediately if animations are not advancing, and a
hard floor settles at 7s. Without those the wordmark sat invisible past 11
seconds in testing.

**2. Sweep for scroll reveals; never use IntersectionObserver alone.**
IO drops entries during fast scrolls and on deep links. Measured on an earlier
build: **22 of 26 sections stayed permanently invisible** after a jump to the
bottom. The fix is a sweep over a shrinking queue, driven directly by the
scroll listener, plus a self-terminating guard interval.

**3. A safety net must not depend on the library it covers for.**
The first version of that guard called `gsap.to()`. GSAP's overwrite logic
cancelled it and **38 of 43 elements stayed hidden**. The guard now writes
plain DOM styles.

**4. `overflow-x: clip`, never `hidden`.**
`hidden` promotes `body` to a scroll container, which silently breaks every
`position: sticky` pin and makes `window.scrollTo` a no-op.

---

## 6. Checking your work

The Claude-in-Chrome extension is unreliable on this machine — it drops tabs
and eventually returns "No tab available". Use headless Chrome instead:

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new \
  --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --virtual-time-budget=15000 --window-size=1440,1000 \
  --screenshot="out.png" "http://localhost:8899/index.html"
```

Gotchas, all of which have produced false "confirmed bugs" here:

- **Use `--headless=new`.** The old `--headless` fires **zero scroll events**,
  so anything scroll-driven looks broken.
- **Anchor URLs do not scroll** under virtual time. Capture a tall window and
  crop, or override the CSS in a throwaway copy.
- **The window clamps to ~497 CSS px minimum**, so a 390px shot is a crop of a
  wider render, not a real mobile layout.
- `--virtual-time-budget` must exceed the intro or the shot catches it mid-run.
- To measure rather than eyeball, inject a script that writes results into
  `document.title`, then read it with `--dump-dom | grep '<title>'`.

Before pushing, confirm at 497 / 768 / 992 / 1408 / 1888 px: no horizontal
overflow (`scrollWidth === clientWidth`), no reveals left hidden, no broken
images, and the boot screen cleared.

---

## 7. Open items

1. **Shoot a hero clip.** Ten seconds of you walking, against a plain light
   wall, phone is fine. That is the one asset standing between this and the
   cinematic reference, whose hero composites a matted person inside the
   letterforms. Right now the hero uses `profile.jpg`, feathered on every edge
   because it is a full scene rather than a cut-out figure.
2. **Replace the PhysioSync screenshot.** Sign in, capture the conditions view
   with the 3D model, save it over `shot-physiosync.jpg`.
3. **Make ShopXtra, PhysioSync and Zaika public** if you are comfortable. That
   is three more code links. Hiring guidance is blunt on this: if a hiring
   manager cannot read your code, your GitHub is invisible.
4. **Move CARE off InfinityFree** if it is the link you send to clients — the
   bot challenge means no link preview and nothing for search engines.

---

## 8. Design history

Every version is in git. To go back to any of them:

```bash
git show <sha>:index.html > index.html
```

Check the asset names afterwards — they changed at `67c3a58`, so older builds
reference PNGs that no longer exist.

| Commit | Design |
|---|---|
| `d4f9eb5` | **Cinematic** — current. Title sequence, scenes, red on black |
| `849a047` | Constellation canvas, custom cursor, sideways rail |
| `33ca0f4` | Terminal CLI with a live prompt, GSAP ScrollTrigger |
| `1553acb` | Arcade HUD — player card, stat bars, level select |
| `3513e60` | Code editor — tab strip, file explorer, `skills.json`, `git log` |
| `f32d97b` | Neobrutalism — bone and ink, hard shadows |

---

© 2026 Hamza Haroon
