# The AI Co-Scientist Typology — static site

Static HTML pages for the nine character cards. Hosted via GitHub Pages.

```
docs/
├── index.html              # landing page with 3×3 grid linking to all cards
├── magician.html           # I — Lapidary (narrow / user-init)
├── temperance.html         # II — Adjuster (narrow / mixed)
├── justice.html            # III — Inspector (narrow / system-init)
├── empress.html            # IV — Drafter (broad / user-init)
├── lovers.html             # V — Collaborator (broad / mixed)
├── wheel.html              # VI — Conductor (broad / system-init)
├── fool.html               # VII — Deputy (whole / user-init)
├── priestess.html          # VIII — Co-Investigator (whole / mixed)
├── hermit.html             # IX — Successor (whole / system-init)
├── cards/                  # the nine card images
├── styles.css
└── manifest.json           # machine-readable card index
```

## How to deploy to GitHub Pages

1. Create a public GitHub repository (or use an existing one).
2. Push the `docs/` directory to the repository.
3. In the repo's **Settings → Pages**, set source to `Deploy from a branch`, branch `main`, folder `/docs`.
4. After a minute or two, the site will be live at `https://{username}.github.io/{repo}/`.

To update content:
1. Edit `CARDS_DATA` in `build_site.py` (one level above `docs/`).
2. Re-run `python build_site.py`.
3. Commit and push.

## How to wire up the Qualtrics survey

Each card lives at a clean URL like `https://{username}.github.io/{repo}/temperance.html`. The Qualtrics survey needs to (a) compute which card the respondent should land on, and (b) redirect there at end-of-survey.

### Step 1: add a Qualtrics embedded-data field

In **Survey Flow**, before the first question, add an `Embedded Data` element with one field:
- `card_slug` (no default value)

### Step 2: compute the card slug at end-of-survey

Add a final hidden question (any type — a Descriptive Text block works) and paste this JavaScript into its **JavaScript** section. The script reads the respondent's selections from question piping (`${q://QID/ChoiceTextEntryValue}`) and writes the resulting card slug into the embedded data field.

See [qualtrics_routing.js](../qualtrics_routing.js) in the repository root for the full script. Adapt the QID references to match your survey's actual question IDs.

### Step 3: configure end-of-survey redirect

In **Survey Options → Survey Termination → Redirect to a URL**, set:

```
https://{username}.github.io/{repo}/${e://Field/card_slug}.html?rid=${e://Field/ResponseID}
```

Substituting `{username}` and `{repo}` with the actual values. The `rid` query parameter carries the Qualtrics response ID through to the result page for tracking.

### Step 4: verify

Take the survey yourself once. At the end, you should be redirected to the correct card page (e.g. `.../temperance.html?rid=R_abc123`). Open the survey response in Qualtrics and confirm the `card_slug` embedded data field was set correctly.

## How the routing logic works

Two axes:
- **Scope** (rows): how much of a task the AI handles. *narrow* → *broad* → *whole task*.
- **Initiative** (columns): who proposes the next move. *user-initiated* → *mixed* → *system-initiated*.

3 × 3 = 9 cells. Inputs:

| Input | Affects | Notes |
|---|---|---|
| Q3 (current uses) | scope | averaged over selected items; count adjustment for many items |
| Q4 (frustrations) | initiative | summed; "overconfident" / "hallucinated" pull toward user-init |
| Q10 (what justifies setup) | scope + initiative | direct demand signal; both axes |
| Q11 (top 3 qualities) | initiative | summed pulls |
| Q1c (competence) | scope + initiative | more competent → narrower + more user-init |
| Q2 (frequency) | scope + initiative | daily users → broader + more system-init tolerance |

Bucket thresholds:
- Scope: `< 1.7` narrow · `1.7–2.3` broad · `> 2.3` whole task
- Initiative: `≤ −1.5` user-init · `−1.5 to +1.5` mixed · `≥ +1.5` system-init

The full Python implementation is in `route_responses.py` (project root). The Qualtrics-side JavaScript in `qualtrics_routing.js` is a translation of the same logic.

## Updating the pilot distribution badges

The red badges on the index page (showing how many of the n=9 pilot respondents land on each card) are hard-coded in the `DISTRIBUTION` dictionary at the top of `build_site.py`. To update:

1. Re-run `route_responses.py` against the latest CSV.
2. Copy the new distribution from the script's output (or from `routing_results.json`) into `DISTRIBUTION` in `build_site.py`.
3. Re-run `build_site.py` and push.

## Citation

If you use or reference this typology, please cite it as:
> Wang, K. *The AI Co-Scientist Typology*. Columbia DBMI, 2026. https://github.com/USERNAME/REPO
