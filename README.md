# Alashá space

Multilingual (RU default, EN; more to come) static site for an independent space for women
writers from Central and North Asia. Hugo extended 0.168, no theme, no Node build.

```sh
hugo server                 # http://localhost:1313/ → redirects to /ru/
hugo --printI18nWarnings    # must finish with no errors or warnings
```

## Where things live

- `content/<lang>/` — one folder per language, identical file paths (that is what links translations).
  Sections: texts, books, interviews, cinema, courses, events, plus `about.md`. Order is by `weight`.
- `i18n/<lang>.toml` — every UI string. Templates contain no user-visible text.
- `layouts/` — `baseof`, `home`, `list`, `single`, `404`; partials in `layouts/_partials/`
  (`home/` for homepage blocks, `cards/` for per-section cards shared by home and list pages).
- `assets/css/main.css` — the designer's stylesheet, **kept verbatim** so new design deliveries can be
  diffed against it. All additions go in `assets/css/hugo.css`.
- `archetypes/` — `hugo new content/ru/texts/foo.md` gets the right front matter.

**Adding a language:** a `[languages.xx]` block in `hugo.toml`, `content/xx/`, `i18n/xx.toml`. No
template changes.

## Gotchas

1. Events use a custom `eventDate` param, not `date`: Hugo hides future-dated pages by default.
2. `resources.Concat` fails on a nil, so the CSS slice is built only from files that exist.
3. Hugo 0.168 language keys are `locale` and `label` (not `languageCode` / `languageName`).
4. `time.Format "Jan"` is localised (`сент`, `окт`); trailing dots are stripped for the design.
5. Inside `range` + `with`, `.` changes: capture the item (`$p := .`) before nesting.
6. Quote all front matter strings (titles contain `«»`, `“”`, `—`, `:`, `·`).
7. A running `hugo server` can get stuck in an error state after large file changes; restart it
   before debugging templates.
8. Headless Chrome won't render narrower than ~500px; check mobile in a real browser.

## Open decisions

Each currently uses the default shown.

| Decision | Current default | Alternatives |
|---|---|---|
| URL scheme | RU default, every language prefixed (`/ru/`, `/en/`) | RU at root; EN default |
| Fonts | System fonts (CSS names Inter but never loads it) | Self-host Inter; a webfont instead of Georgia |
| Newsletter | Demo form, not wired up | Buttondown, Mailchimp, Substack |
| Courses / payments | Price as text, placeholder course pages | Teachable, Gumroad, Podia |
| Events ordering | By `weight` | By `eventDate`, upcoming/past split |
| Translator workflow | Edit Markdown in git | Decap CMS, CloudCannon, Pages CMS |
| Hosting | Not set up | Netlify, Cloudflare Pages, GitHub Pages |
| Domain / email | `alasha.space`, `hello@alasha.space` (from the prototype) | Confirm |
| Tags / categories | Disabled | Enable with styled templates |

## Not yet verified

- Mobile (≤580px) in a real browser: Menu toggle, single-column grids, no horizontal scroll.
