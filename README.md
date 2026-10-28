# TUPA Lab website

The website for **TUPA (Transport and Urban Planning with AI)**, a research lab at KAIST's Cho Chun Shik Graduate School of Mobility. Built during my internship at the lab in 2026.

- **Live:** https://tupa-two.vercel.app
- **Stack:** Next.js 14 (App Router), TypeScript, Tailwind CSS, Notion as the CMS, deployed on Vercel
- **Replaces:** the lab's old WordPress site

The professor, who isn't a developer, edits members, publications, research areas, projects, openings, news and the gallery in Notion tables. The site updates within an hour, or right away from a refresh link.

> This is a copy of [yunhalee2008/TUPA](https://github.com/yunhalee2008/TUPA), the repo Vercel deploys from.

## Architecture

```
pages (app/*) ──▶ lib/content.ts   the only data surface: async getters
                       │
            Notion first, when NOTION_API_KEY is set
                       ▼
                lib/notion.ts      direct REST calls, ISR (1 h), falls back on failure
                       │ fallback
                       ▼
            built-in data          content.ts constants + lib/data/*.json
```

- **Always up.** If Notion is empty or down, the site renders from the built-in data.
- **Hybrid content.** Members, publications, research areas, projects, openings and settings have Notion as their single source (fully seeded). News and the gallery merge the legacy archive in code with new Notion entries.
- **Korean / English toggle.** Both languages render on every page; a `html[data-lang]` attribute and CSS show one (`.ko-only` / `.en-only`, see `components/LangToggle.tsx`).
- **Caching.** Every Notion fetch uses `revalidate: 3600` and the `"notion"` tag. `/api/refresh?token=REVALIDATE_TOKEN` invalidates it immediately.

## Development

```bash
npm install
cp .env.local.example .env.local   # fill in NOTION_API_KEY and REVALIDATE_TOKEN
npm run dev                         # http://localhost:3000
npm run build && npm run lint       # before every commit
```

- It runs without any keys, on the built-in data. Keys are only needed to test the Notion integration.
- Don't run `npm run build` while the dev server is running; it corrupts `.next`. Use `NEXT_DIST_DIR=.next-build npm run build` for an isolated build.

## Environment variables

| Variable | Purpose |
|---|---|
| `NOTION_API_KEY` | Notion internal integration secret; the "TUPA CMS" page must be connected to it |
| `REVALIDATE_TOKEN` | Token for the `/api/refresh` instant refresh (any string) |
| `NOTION_DB_*` | Optional database ID overrides; defaults are in `lib/notion.ts` |

## Layout

- `lib/content.ts`: the data contract (types and getters). Pages import only this.
- `lib/notion.ts`: the Notion fetch layer.
- `lib/data/`: legacy archive data (news bodies and image index, old news, gallery, research overview).
- `public/people|news|gallery|projects/`: images migrated from the old site. New images are uploaded to Notion.
- `public/hero/`: the hero video.
- `scripts/`: one-off migration scripts (Notion seed dump, member photo upload).
- `docs/`: CMS schema and design-system notes.

## Changing the domain

1. Connect the domain in Vercel and set up DNS
2. Update `SITE_URL` in `app/layout.tsx`, plus `app/sitemap.ts` and `app/robots.ts`
3. Replace the refresh link URL in the Notion guide
4. Register the site in Google Search Console
