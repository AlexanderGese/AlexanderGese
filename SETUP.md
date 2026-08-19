# SETUP

Install order matters a little — the Actions need to run once before three
sections of the README stop showing broken images.

---

## 1. Files

```
AlexanderGese/                    ← the repo named after your username
├── README.md
└── .github/
    └── workflows/
        ├── snake.yml
        ├── 3d-contrib.yml
        ├── activity.yml
        └── papers.yml
```

Then: **Settings → Actions → General → Workflow permissions →
"Read and write permissions"**. Without this every workflow fails on push.

Go to the **Actions** tab and hit *Run workflow* on all four. Snake and the
3D calendar take ~1 min; the other two are near-instant.

---

## 2. Kill the rate limiting (the actual fix — 4 minutes)

Two cards in `DIAGNOSTICS` read the GitHub API. The public
`github-readme-stats.vercel.app` runs on one shared token for hundreds of
thousands of profiles, so it spends a good chunk of every day returning
*"Downtime due to GitHub API rate limiting."* That's what you were seeing.
Your own instance gets its own token and its own 5,000 req/hr budget.

1. Fork <https://github.com/anuraghazra/github-readme-stats>
2. Create a classic PAT at <https://github.com/settings/tokens> — tick
   **`repo`** if you want private-repo commits counted, otherwise no scopes
   at all is fine. Set it to **no expiry**.
3. Go to <https://vercel.com>, *Add New → Project*, import your fork,
   **Deploy**.
4. In the Vercel project: *Settings → Environment Variables* → add
   `PAT_1` = your token. Then *Deployments → ⋯ → Redeploy*.
5. Copy the hostname it gives you (e.g. `alexander-stats.vercel.app`).
6. In `README.md`, find/replace **`STATS_HOST`** → that hostname. Four hits.

Sanity check: `https://<your-host>/api/status/pat-info` should report your
token as healthy. If a card ever goes blank again, that URL tells you why —
almost always an expired token.

> The same trick works for the streak card
> (`DenverCoder1/github-readme-streak-stats`) if you want it back. I left it
> out because the streak number is the single most fragile widget on GitHub
> and it looked bad on a profile otherwise built to not break.

---

## 3. What's tied to what

| README section | Source | Can it rate-limit? |
|---|---|---|
| Header, typing line, footer | capsule-render, readme-typing-svg | No — no GitHub API |
| `ENVIRONMENT` icons | skillicons.dev, shields.io | No |
| Follower / view badges | shields.io, komarev | No |
| `DIAGNOSTICS` cards | **your** Vercel instance | No, once step 2 is done |
| Trophies | github-profile-trophy | Occasionally — it's the one remaining hotlink |
| Snake | `snake.yml` → `output` branch | No |
| 3D calendar | `3d-contrib.yml` → `main` | No |
| Recent activity | `activity.yml` → README | No |
| Reading pile | `papers.yml` → README | No |

---

## 4. Troubleshooting

**Snake / 3D calendar still broken.** They 404 until their first successful
run. Check the Actions tab for a red X — nine times out of ten it's the
write-permission setting in step 1.

**Activity / reading pile stay empty.** The HTML comment markers must survive
verbatim, including case. If you reflow the README, don't touch these:

```
<!--START_SECTION:activity-->     <!--END_SECTION:activity-->
<!-- BLOG-POST-LIST:START -->     <!-- BLOG-POST-LIST:END -->
```

**Snake shows a mostly-empty grid.** `Platane/snk` reads the public
contribution graph. Turn on *Settings → Profile → Contributions →
Include private contributions* if you want private commits in there.

**Trophies vanish.** Only truly optional thing here. Delete the block or
self-host `ryo-ma/github-profile-trophy` the same way as step 2.

**arXiv feed is too noisy.** `cs.RO` and `cs.LG` move fast. Narrower options:
`cs.AI`, `cs.NE`, `stat.ML`. Or point `feed_list` at your own blog once the
research writing goes public.
