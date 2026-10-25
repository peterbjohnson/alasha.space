# Alashá space — Build Requirements (v1 "hello world")

Status: **v1 built** (2026-10-25) at the repo root. This document is the brief it was built from. A throwaway
prototype build was done in `quickstart/` (to be deleted). Everything worth keeping from it is
captured here. Nothing in this document depends on that folder.

---

## 1. Goal and scope

Build a static, multilingual Hugo site for **Alashá space**, an independent space for women
writers from Central and North Asia. It has a journal (texts), book recommendations, interviews,
film criticism, writing courses, events, a call for submissions and a newsletter sign-up.

**v1 ("hello world") is done when:**

- The homepage looks like the design prototype (`design/prototype-v01/index.html`) in **both
  Russian and English**.
- Every card on the homepage links to a real (placeholder) page.
- Every section has a list page, every item has a single page, and there is an About page and a 404.
- A language switcher moves between the same page in RU and EN.
- `hugo` builds with no errors, no warnings and no missing translations
  (`hugo --printI18nWarnings`).

**Out of scope for v1:** real content, a newsletter provider, payments/course platform,
CMS, analytics, deployment pipeline, search, tags/categories.

---

## 2. Source material

| Path | What it is |
|---|---|
| `design/requirements_v01.zip` | Original delivery (kept unmodified for reference) |
| `design/prototype-v01/index.html` | One-page static prototype, all copy in Russian |
| `design/prototype-v01/styles.css` | Complete stylesheet (design tokens in `:root`, responsive at 900px / 580px) |
| `design/prototype-v01/script.js` | Mobile menu toggle (10 lines) |
| `design/prototype-v01/assets/logo.png` | Logo, 1024×1024 PNG, 1.4 MB, with transparent padding |
| `design/prototype-v01/README.txt` | Note from the designer (Russian; translated below) |

**README.txt (translated):**
> Alashá space — static site prototype.
> Open: index.html in a browser.
> Files: index.html, styles.css, script.js, assets/logo.png.
> Before publishing, replace the demo texts, links, prices, email and event dates with real ones.

The prototype is the **visual source of truth**. Port its HTML structure and CSS class names
as-is. Do not redesign.

---

## 3. Tech stack

- **Hugo extended v0.168.0** (installed via Homebrew). Uses the new template system
  (Hugo ≥ 0.146): templates live directly in `layouts/` (`baseof.html`, `home.html`, `list.html`,
  `single.html`, `404.html`). Partials go in `layouts/_partials/`.
- **No theme.** Ananke was tried and fully overridden. It contributed nothing except a stray sample
  image and confusing fallbacks. Templates live in the project's `layouts/`.
  (Optional later: move them into `themes/alasha/` if a reusable theme is ever wanted.)
- **No Node/npm build.** Hugo Pipes handles CSS/JS (concat, minify, fingerprint) and images
  (resize, WebP).
- Git repository at the project root, default branch **`main`**.

---

## 4. Project structure

```
alasha/
├── hugo.toml
├── .gitignore                 # public/, resources/_gen/, .hugo_build.lock, quickstart/
├── README.md
├── docs/REQUIREMENTS.md       # this file
├── design/                    # reference only, never published
│   ├── requirements_v01.zip
│   └── prototype-v01/
├── archetypes/                # one archetype per content type (texts.md, books.md, …)
├── assets/
│   ├── css/main.css           # prototype styles.css, copied verbatim
│   ├── css/hugo.css           # additions only (lang switch, list/single pages, active nav)
│   ├── js/main.js             # prototype script.js, copied verbatim
│   └── images/logo.png        # prototype logo (processed by Hugo, never served raw)
├── content/
│   ├── ru/                    # Russian content (default language)
│   └── en/                    # English content, same file paths as ru/
├── data/                      # (unused in v1)
├── i18n/
│   ├── ru.toml
│   └── en.toml
├── layouts/
│   ├── baseof.html
│   ├── home.html
│   ├── list.html
│   ├── single.html
│   ├── 404.html
│   └── _partials/
│       ├── header.html
│       ├── footer.html
│       ├── logo.html
│       ├── lang-switch.html
│       ├── ornament.html
│       ├── section-head.html
│       ├── event-list.html
│       ├── newsletter.html
│       └── home/{texts,books,interview,cinema,courses,events}.html
└── static/                    # (empty in v1)
```

---

## 5. Configuration (`hugo.toml`)

Known-good config (verified on Hugo 0.168). Note the new language keys **`locale`** and
**`label`** (not `languageCode` / `languageName`).

```toml
baseURL = 'https://alasha.space/'
title = 'Alashá space'

defaultContentLanguage = 'ru'
defaultContentLanguageInSubdir = true      # every language under its own prefix: /ru/, /en/
disableKinds = ['taxonomy', 'term']        # no tags/categories in v1

[languages]
  [languages.ru]
    locale = 'ru-RU'
    label = 'RU'
    title = 'Alashá space'
    contentDir = 'content/ru'
    weight = 1
    [languages.ru.params]
      description = 'Alashá space — пространство для пишущих женщин из Центральной и Северной Азии.'
  [languages.en]
    locale = 'en-GB'
    label = 'EN'
    title = 'Alashá space'
    contentDir = 'content/en'
    weight = 2
    [languages.en.params]
      description = 'Alashá space — a space for women writers from Central and North Asia.'

[params]
  email = 'hello@alasha.space'
  [[params.social]]
    name = 'Instagram'
    url = '#'
  [[params.social]]
    name = 'Telegram'
    url = '#'
  [[params.social]]
    name = 'YouTube'
    url = '#'
```

---

## 6. Multilingual requirements

- Languages now: **RU (default)** and **EN**. More will follow, so nothing may hard-code two languages.
- **Adding a language** must need only: a `[languages.xx]` block, `content/xx/`, `i18n/xx.toml`.
  No template changes.
- **Translation by directory:** `content/ru/texts/foo.md` and `content/en/texts/foo.md` are
  linked automatically because they share a path. Slugs/filenames are English and identical
  across languages.
- **UI strings** (buttons, labels, aria labels, footer) live in `i18n/*.toml` and are used with
  `i18n "key"`. **Page copy** lives in content front matter/body. **No user-visible text in
  templates.**
- `<html lang="{{ site.Language.Locale }}">`.
- `<link rel="alternate" hreflang>` for every translation of the current page.
- **Language switcher** in the header: shows every language's `label`. The current one is not a
  link (`aria-current`). The others link to the translated page, or to that language's home if
  there is no translation. Iterate `hugo.Sites` (not a hard-coded list).
- The root URL `/` redirects to `/ru/` (Hugo does this automatically with the config above).
- Dates are localised with `time.Format` (Hugo uses the language locale). Russian short months come
  out with a trailing dot or as `сент`; strip the trailing `.` for the design.
- **The English site must be fully translated.** No Russian may appear on any `/en/` page.
  See §13 for the complete copy deck.

---

## 7. Content model

Six sections plus About. Each section `_index.md` carries the homepage block heading and
registers itself in the main menu:

```yaml
---
title: "Texts"            # menu label + list page title
eyebrow: "Journal"        # small uppercase label above heading
heading: "New texts"      # big heading on homepage block / list page
menus:
  main:
    weight: 10
---
```

Menu weights: texts 10, books 20, interviews 30, cinema 40, courses 50, events 60.

Ordering within a section: **`weight`** (homepage shows the first 3 by weight).

Front matter per item type (all items also have `title`, `weight`, and a placeholder body):

| Section | Fields | Notes |
|---|---|---|
| `texts` | `tag`, `region`, `minutes`, `summary` | First item renders as the large wine-coloured card |
| `books` | `kicker`, `coverLetter`, `coverStyle` (`a`/`b`/`c`), `summary` | Cover is a coloured block with one letter |
| `interviews` | `kicker`, `minutes`, `summary` | Homepage shows only the first interview |
| `cinema` | `summary` | Numbered 01–03 automatically |
| `courses` | `kicker`, `price` (string, e.g. `"£49"`), `featured` (bool), `summary` | Featured card is wine-coloured with a light button |
| `events` | `kicker`, `eventDate` (`YYYY-MM-DD`), `eventTime` (string, e.g. `"19:00 GMT+5"`) | **Use `eventDate`, not `date`.** Hugo hides future-dated pages by default |

Home page `content/<lang>/_index.md` front matter: `title`, `eyebrow`, `heading` (may contain
`<em>`, so render with `safeHTML`), `lead`, `ctaPrimary`, `ctaSecondary`, `noteMark`, `noteTitle`,
`noteText`, `noteLink`, `submitEyebrow`, `submitTitle`, `submitText`, `submitFine`,
`newsletterEyebrow`, `newsletterTitle`.

`about.md`: `title` and body.

Front matter must be valid YAML. Quote all strings, since titles contain `«»`, `“”`, `—`, `:` and `·`.

Each section needs an archetype in `archetypes/` with the fields above, so `hugo new` produces
the right front matter.

---

## 8. Page templates

### baseof.html
- Meta: charset, viewport, description (page `description`, else `summary`, else site
  `params.description`).
- Title: home = page title; other pages = `Page — Alashá space`.
- CSS: `main.css` + `hugo.css`, merged with `resources.Concat`, then minified and fingerprinted
  with SRI. **Build the slice only from resources that exist** (see §12, gotcha 1).
- Favicon: logo cropped/resized to 64×64 PNG via Hugo image processing.
- JS: `main.js` minified and fingerprinted, at the end of `<body>`.
- Structure: `header` partial → `<main id="top">{{ block "main" }}</main>` → `footer` partial.

### Header (`site-header`)
Logo (links to current language home) · "Menu" toggle button (mobile) · main nav from
`site.Menus.main` (active item gets `aria-current="page"`) · language switcher · "Submit a text"
pill linking to `<home>#submit`.

### Homepage (in this order, matching the prototype)
1. **Hero:** eyebrow, h1, lead, two buttons (`#texts`, `#courses`), plus the "Theme of the month"
   note aside linking to the texts section.
2. **Ornament divider** (pure CSS, `aria-hidden`).
3. **Texts:** `feature-grid`, 3 cards, the first `large`.
4. **Books:** `band`, `book-grid`, 3 books.
5. **Interview:** `split`, decorative `portrait-art` (role=img + aria-label), first interview.
6. **Cinema:** dark `cinema-section`, `section-head light`, 3 numbered columns.
7. **Courses:** `course-grid`, 3 cards (`featured` vs `pale`).
8. **Events:** `band warm`, `event-list` (shared partial with the events list page).
9. **Submit:** `#submit`, open-call copy, `mailto:` button using `params.email`.
10. **Newsletter:** wine band, email form (demo only, see §10).

Each section block uses a shared `section-head` partial (eyebrow + heading + optional
"All … →" link to the section list page).

### list.html
Section head, then a 3-column card grid of all items by weight. Events use the `event-list` partial.
Must collapse to 1 column at ≤900px.

### single.html
Narrow column (`max-width: 760px`): eyebrow linking back to the section, serif h1, meta line
(region · minutes, when present), summary as the lead, body.

### 404.html
Eyebrow "404", heading, link back home. All text from i18n.

### Footer
Logo + tagline · links (Journal → texts, Courses, For authors → `#submit`, About) · social links
from `params.social` · `© {{ now.Year }} Alashá space`.

---

## 9. Styling and assets

- Copy `styles.css` to `assets/css/main.css` **unchanged**, so it can be diffed against future
  design deliveries. Anything new goes in `assets/css/hugo.css`:
  - `.lang-switch` (small, bold, letter-spaced; current language in `--wine`; `margin-left:auto`
    on ≤900px so it sits next to the Menu button)
  - `.nav a[aria-current]` in `--wine`
  - `.list-grid` (3 columns → 1 at ≤900px)
  - `.prose` (single page column, serif h1 `clamp(40px,5vw,64px)`, serif 19px body)
- Logo: never serve the 1.4 MB original. Use `.Resize "400x webp"` (≈24 KB) with explicit
  width/height. The header and footer both use one `logo.html` partial. The prototype CSS scales the
  logo up (`transform: scale(1.38)` / `1.4`) to crop its transparent padding. Keep that.
- Fonts: the CSS lists `Inter` but never loads it, so it currently falls back to system fonts
  (open decision, §14).

---

## 10. Behaviour

- `main.js`: toggles `.nav.open` and `aria-expanded` on the Menu button. Closes the menu when a
  link is clicked.
- Newsletter form: demo only, as in the prototype (`onsubmit` prevents submission and adds `.sent`,
  which shows the success message). Mark it clearly in a template comment as not wired up.

---

## 11. Accessibility and SEO

- Single `h1` per page. Section headings `h2`, cards `h3`.
- `aria-label` on brand link, main nav, language switcher, decorative illustration (from i18n).
- The email input has a visually-hidden `<label>` (`.sr-only`).
- `<time datetime="YYYY-MM-DD">` on events.
- `hreflang` alternates (§6). Canonical = page permalink.
- Hugo's built-in sitemap is fine for v1.

---

## 12. Lessons learned from the quickstart attempt

1. **`resources.Concat` fails on a nil.** If either CSS file is missing, the whole site 500s with
   `expected slice of Resource objects, received []interface {}`. Build the slice with
   `range … with resources.Get` so missing files are skipped.
2. **A running `hugo server` can get stuck** in an error state if files are added while it runs
   (it rebuilt mid-write and kept the error). If pages 500 after big changes, restart the server
   before debugging templates.
3. **Ananke adds nothing** for a custom design like this. Overriding its `baseof.html` overrides
   everything anyway. Go theme-less.
4. **Hugo 0.168 language keys** are `locale` and `label`. `hugo config` shows no deprecation
   warnings with the config in §5.
5. **Future-dated content is unpublished by default.** Events need a custom `eventDate` param.
6. **`time.Format "Jan"` is localised** (`сент`, `окт`, `Sept`). Strip any trailing `.`.
7. **Menus from front matter** (`menus: main` in section `_index.md`) give per-language menus
   with no config duplication.
8. **Inside `range` + `with`, `.` changes.** Capture the item (`{{ $p := . }}`) before nesting
   `with`. A bug in the first list template read `minutes` from the section, not the item.
9. **Generating front matter:** emit strings via JSON encoding (valid YAML) to avoid quote breakage.
10. **Visual checks:** headless Chrome will not render narrower than about 500px, so `--window-size=390,…`
    screenshots look clipped. Check mobile in a real browser or with device emulation.
11. The original prototype has the same mobile behaviour, so compare against it before "fixing" CSS.

---

## 13. Copy deck (RU → EN)

All Russian in the prototype, with the English to use. The **English column is the source for
`content/en/` and `i18n/en.toml`**. Russian stays as-is in `content/ru/` and `i18n/ru.toml`.

### 13.1 Site meta

| Key | RU | EN |
|---|---|---|
| home `title` | Alashá space — тексты, книги, кино и письмо | Alashá space — texts, books, cinema and writing |
| `params.description` | Alashá space — пространство для пишущих женщин из Центральной и Северной Азии. | Alashá space — a space for women writers from Central and North Asia. |

### 13.2 UI strings (`i18n/*.toml`)

| Key | RU | EN |
|---|---|---|
| `home` (brand aria suffix) | на главную | home |
| `menu` | Меню | Menu |
| `mainNav` | Основная навигация | Main navigation |
| `languageSwitch` | Язык | Language |
| `submitText` | Прислать текст | Submit a text |
| `read` | Читать → | Read → |
| `readText` | Читать текст → | Read the text → |
| `readInterview` | Читать интервью | Read the interview |
| `details` | Подробнее → | Details → |
| `aboutCourse` | О курсе | About the course |
| `minutes` | мин | min |
| `allTexts` | Все публикации → | All publications → |
| `allBooks` | Все рекомендации → | All recommendations → |
| `allCinema` | Все колонки → | All columns → |
| `allCourses` | Все программы → | All programmes → |
| `journal` | Журнал | Journal |
| `authors` | Авторам | For authors |
| `about` | О проекте | About |
| `subscribe` | Подписаться | Subscribe |
| `emailPlaceholder` | ваш@email.com | your@email.com |
| `subscribed` | Спасибо! Вы в списке. | Thank you! You are on the list. |
| `footerTagline` | Независимое пространство для текста, разговора и обучения. | An independent space for writing, conversation and learning. |
| `decorative` | Декоративная иллюстрация | Decorative illustration |
| `notFound` | Страница не найдена | Page not found |
| `backHome` | Вернуться на главную → | Back to the homepage → |

### 13.3 Homepage (`content/<lang>/_index.md`)

| Field | RU | EN |
|---|---|---|
| `eyebrow` | Женские голоса Центральной и Северной Азии | Women's voices of Central and North Asia |
| `heading` | Место, где письмо становится `<em>`пространством`</em>` | A place where writing becomes `<em>`space`</em>` |
| `lead` | Рассказы, эссе, интервью, книжные рекомендации и разговоры о кино. А ещё — практические курсы для тех, кто хочет писать точнее, смелее и свободнее. | Stories, essays, interviews, book recommendations and conversations about film. And practical courses for those who want to write more precisely, boldly and freely. |
| `ctaPrimary` | Читать новое | Read the latest |
| `ctaSecondary` | Учиться письму | Learn to write |
| `noteMark` | № 01 | No. 01 |
| `noteTitle` | Тема месяца | Theme of the month |
| `noteText` | Дом как память: что мы наследуем, что берём с собой и что создаём заново. | Home as memory: what we inherit, what we take with us and what we create anew. |
| `noteLink` | Подборка текстов → | Read the selection → |
| `submitEyebrow` | Open call | Open call |
| `submitTitle` | У вас есть текст? | Have a text? |
| `submitText` | Мы принимаем рассказы, эссе, колонки и экспериментальные формы от пишущих женщин, связанных с Центральной и Северной Азией. | We accept stories, essays, columns and experimental forms from women writers connected to Central and North Asia. |
| `submitFine` | Пожалуйста, приложите короткую биографию и текст в .docx или Google Docs. | Please attach a short bio and your text as .docx or Google Docs. |
| `newsletterEyebrow` | Письма Alashá | Alashá letters |
| `newsletterTitle` | Раз в две недели — лучшее из журнала и новые анонсы | Every two weeks — the best of the journal and new events |

### 13.4 Sections (`content/<lang>/<section>/_index.md`)

| Section | `title` RU / EN | `eyebrow` RU / EN | `heading` RU / EN |
|---|---|---|---|
| texts | Тексты / Texts | Журнал / Journal | Новые тексты / New texts |
| books | Книги / Books | Книжная полка / Bookshelf | Что читать / What to read |
| interviews | Интервью / Interviews | Интервью / Interview | Интервью / Interviews |
| cinema | Кино / Cinema | Кинокритика / Film criticism | Смотреть внимательнее / Watch more closely |
| courses | Курсы / Courses | Alashá school / Alashá school | Курсы по письму / Writing courses |
| events | Анонсы / Events | Анонсы / Events | Встречи и события / Meetings and events |

### 13.5 Items

Placeholder body for every item: RU «Демонстрационный текст. Замените на настоящую публикацию.» /
EN "Placeholder text. Replace with the real publication."

**Texts**

| File | Field | RU | EN |
|---|---|---|---|
| `grandmother-language.md` (w1) | title | Язык, на котором бабушка молчала | The language my grandmother kept silent in |
| | tag | Эссе | Essay |
| | region · minutes | Кыргызстан · 12 | Kyrgyzstan · 12 |
| | summary | О семейной памяти, переводе и словах, которые возвращаются через поколения. | On family memory, translation and words that return across generations. |
| `last-bus-to-the-steppe.md` (w2) | title | Последний автобус в степь | The last bus to the steppe |
| | tag | Рассказ | Story |
| | region · minutes | Казахстан · 8 | Kazakhstan · 8 |
| | summary | Ночная дорога, две незнакомки и разговор, после которого не получается жить по-старому. | A night road, two strangers and a conversation after which life can't go on as before. |
| `snow-is-an-archive.md` (w3) | title | Снег — это тоже архив | Snow is an archive too |
| | tag | Колонка | Column |
| | region · minutes | Якутия · 6 | Yakutia · 6 |
| | summary | Заметки о ландшафте, памяти и северной оптике. | Notes on landscape, memory and a northern way of seeing. |

**Books**

| File | Field | RU | EN |
|---|---|---|---|
| `novel-of-memory-and-road.md` (w1, cover a) | title | Роман о памяти и дороге | A novel of memory and the road |
| | kicker | Редакция рекомендует | Editors' pick |
| | coverLetter | А | A |
| | summary | Почему мы возвращаемся к книгам, где география становится внутренним состоянием. | Why we return to books where geography becomes an inner state. |
| `five-books-on-friendship.md` (w2, cover b) | title | 5 книг о женской дружбе | 5 books on female friendship |
| | kicker | Голос читательницы | Reader's voice |
| | coverLetter | Ж | F |
| | summary | От семейных саг до короткой прозы — без идеализации и привычных клише. | From family sagas to short prose — without idealisation or the usual clichés. |
| `how-translation-changes-the-view.md` (w3, cover c) | title | Как перевод меняет оптику | How translation changes the view |
| | kicker | Новая переводная проза | New prose in translation |
| | coverLetter | С | T |
| | summary | Разговор о том, какие тексты путешествуют между языками — и что меняется по дороге. | A conversation about which texts travel between languages — and what changes on the way. |

**Interviews**

| File | Field | RU | EN |
|---|---|---|---|
| `right-to-complexity.md` (w1) | title | «Писать — значит возвращать себе право на сложность» | “To write is to reclaim the right to complexity” |
| | kicker · minutes | Большое интервью · 18 | Long interview · 18 |
| | summary | Разговор с писательницей и переводчицей о многоязычии, женских архивах, региональном опыте и литературе вне центра. | A conversation with a writer and translator about multilingualism, women's archives, regional experience and literature beyond the centre. |

**Cinema**

| File | RU title / summary | EN title / summary |
|---|---|---|
| `whose-gaze.md` (w1) | Кому принадлежит взгляд? / Разбираем женскую субъектность в современном кино региона. | Who owns the gaze? / Examining female subjectivity in the region's contemporary cinema. |
| `steppe-as-character.md` (w2) | Степь как персонаж / Почему пейзаж в кино никогда не бывает просто красивым фоном. | The steppe as a character / Why landscape on film is never just a pretty backdrop. |
| `after-the-credits.md` (w3) | После титров / Короткая колонка о фильме, который хочется обсуждать ещё долго. | After the credits / A short column about a film you want to keep talking about. |

**Courses**

| File | Field | RU | EN |
|---|---|---|---|
| `personal-essay.md` (w1, featured, £49) | title | Личный текст: от наблюдения к эссе | The personal text: from observation to essay |
| | kicker | Видеокурс · 8 уроков | Video course · 8 lessons |
| | summary | Как найти тему, удержать голос и собрать материал в цельный текст — без академической тяжести. | How to find a subject, hold your voice and shape material into a whole — without academic heaviness. |
| `dialogue-that-sounds.md` (w2, £35) | title | Диалог, который звучит | Dialogue that rings true |
| | kicker | Видеокурс · 6 уроков | Video course · 6 lessons |
| | summary | Практика для прозы: речь, ритм, подтекст и характер. | Practice for prose: speech, rhythm, subtext and character. |
| `edit-without-fear.md` (w3, £24) | title | Редактировать без страха | Editing without fear |
| | kicker | Мини-курс · 4 урока | Mini-course · 4 lessons |
| | summary | Как сокращать, перестраивать и усиливать собственный текст. | How to cut, restructure and strengthen your own text. |

**Events** (dates/times are demo values from the prototype; year assumed 2026)

| File | eventDate · eventTime | RU title / kicker | EN title / kicker |
|---|---|---|---|
| `writing-about-family.md` (w1) | 2026-09-24 · 19:00 GMT+5 | Как писать о семье и сохранять границы / Онлайн · открытая встреча | Writing about family while keeping boundaries / Online · open meeting |
| `book-club-new-prose.md` (w2) | 2026-10-03 · 18:30 GMT+6 | Читаем новую прозу женщин Центральной Азии / Книжный клуб | Reading new prose by women of Central Asia / Book club |
| `landscape-power-memory.md` (w3) | 2026-10-12 · 20:00 GMT+5 | Пейзаж, власть и память в современном кино / Разговор о кино | Landscape, power and memory in contemporary cinema / Film conversation |

**About** (`about.md`)

| RU | EN |
|---|---|
| title: О проекте | title: About |
| Alashá space — независимое пространство для текста, разговора и обучения. + placeholder | Alashá space is an independent space for writing, conversation and learning. + placeholder |

---

## 14. Open decisions

Each has a default the build will use unless decided otherwise.

| # | Decision | Default for v1 | Alternatives |
|---|---|---|---|
| 1 | Default language and URL scheme | RU default, all languages prefixed (`/ru/`, `/en/`), `/` redirects to `/ru/` | RU at root without prefix (`defaultContentLanguageInSubdir = false`); EN default |
| 2 | Theme | None (project `layouts/`) | Package as `themes/alasha/` |
| 3 | Fonts | System fonts (as the prototype actually renders) | Self-host Inter; pick a webfont for the serif instead of Georgia |
| 4 | Newsletter | Demo form, no provider | Buttondown, Mailchimp, Substack embed |
| 5 | Courses / payments | Price shown as text, "About the course" goes to a placeholder page | Link out to Teachable / Gumroad / Podia |
| 6 | Events ordering | By `weight` | By `eventDate`, with upcoming/past split |
| 7 | Editing workflow for translators | Edit Markdown in git | Decap CMS / CloudCannon / Pages CMS |
| 8 | Hosting | Not set up in v1 | Netlify, Cloudflare Pages, GitHub Pages |
| 9 | Domain and email | `alasha.space`, `hello@alasha.space` (from the prototype) | Confirm |
| 10 | Tags / categories | Disabled | Enable with styled templates |

---

## 15. Acceptance checklist

- [ ] `hugo` and `hugo --printI18nWarnings` complete with zero errors and warnings.
- [ ] `/` redirects to `/ru/`. `/ru/` and `/en/` homepages render all 10 blocks in prototype order.
- [ ] Side-by-side with `design/prototype-v01/index.html` at desktop width, the RU homepage is
      visually equivalent.
- [ ] No Cyrillic text anywhere under `public/en/` (check: `grep -rlP '[\x{0400}-\x{04FF}]' public/en`
      returns nothing).
- [ ] All 6 section list pages and all 16 item pages exist in both languages, plus About and 404.
- [ ] Language switcher on any page goes to the same page in the other language.
- [ ] Main menu highlights the current section.
- [ ] Mobile (real browser, ≤580px): Menu button toggles nav, grids collapse to one column, no
      horizontal scroll.
- [ ] Logo served as WebP ≤ 50 KB. No request for the 1.4 MB PNG.
- [ ] CSS and JS fingerprinted with SRI.
- [ ] No user-visible strings in templates (all via `i18n` or front matter).
- [ ] Adding a dummy third language (`[languages.kk]` + empty `content/kk` + `i18n/kk.toml`)
      needs no template edits.

---

## 16. After v1 (future)

Real content and imagery for cards (`coverImage` per item via page bundles); course detail pages
with curriculum; event detail pages with registration links; RSS per section; search (Pagefind);
Open Graph images; analytics; CI deploy; Kazakh / Kyrgyz / other languages.
