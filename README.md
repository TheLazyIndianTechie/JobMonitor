# IGG Job Monitor

IGG Job Monitor (MWAP) finds videogame-industry job vacancies in India and shares the
good ones with the right WhatsApp Community groups.

**Docs:** [project status and roadmap](IGG_MWAP_Project_Status_and_Roadmap_20_Nov_2026.md) ·
[documentation site](https://igg-job-monitor-tau.vercel.app)

For each vacancy it:

1. **Discovers** listings from studios, publishers and industry service providers.
2. **Filters** them by employer, role and India eligibility.
3. **Verifies** that the vacancy is genuine and still open.
4. **Removes duplicates** and keeps a record of where each listing came from.
5. **Routes** the job to the matching Community group by discipline.
6. **Waits for a human to approve** it before anything is shared.

## How it is built

- **Google Sheets** is the database: jobs, sources, employers, group mappings, the
  approval queue and logs.
- **Google Apps Script** is the automation engine that runs the steps above.
- **WhatsApp posting** is planned but not built. The option being tested is WAHA, an
  unofficial self-hosted WhatsApp Web gateway.

## Status

The project is in development and validation. It is not live.

- The approval workflow has passed controlled end-to-end tests.
- Discovery, verification, duplicate checks and the database writer are developed and
  tested, but no genuine vacancies are being collected or inserted.
- Nothing is posted to WhatsApp yet. The next milestone is a local, read-only WAHA
  feasibility test.

## Documentation

- [Project status and roadmap (20 Nov 2026)](IGG_MWAP_Project_Status_and_Roadmap_20_Nov_2026.md):
  full architecture, test results, safety controls, next steps and the open questions
  for Community administrators.

## Deploying

The landing page in `public/index.html` is hosted on Vercel at
https://igg-job-monitor-tau.vercel.app. Only the `public/` folder is served; the
Markdown docs are not published.

**Automatic (the normal way).** No commands needed beyond Git:

```bash
git checkout -b my-change
git commit -am "Describe the change"
git push -u origin my-change     # Vercel builds a preview URL for the branch
```

Open a pull request, then merge it into `master`. Every merge to `master` deploys to
production.

**Manual.** For members of the Vercel team `lateprofessors-projects`:

```bash
npm i -g vercel                  # once
vercel login                     # once
vercel link --scope lateprofessors-projects --project igg-job-monitor   # once per clone
vercel                           # preview deployment
vercel --prod                    # production deployment
```

To preview the page locally, open `public/index.html` in a browser. There is no build
step.

This repository currently holds documentation only. The Apps Script code and the
Google Sheet live outside it.

## License

[MIT](LICENSE)
