# transcend library

The Transcend Library: community-written guides to trans healthcare, transition and
support in Singapore. Built with [Jekyll](https://jekyllrb.com) from the library's original
Notion archives, on a customised copy of
[jekyll-theme-console](https://github.com/b2a3e8/jekyll-theme-console) (MIT, see
`THEME-LICENSE.txt`).

---

## Contents

- [Editing the site](#editing-the-site)
- [Page settings (front matter)](#page-settings-front-matter)
- [Running it locally](#running-it-locally)
- [Deploying](#deploying)
- [Bringing in pages from Notion](#bringing-in-pages-from-notion)
- [Link previews (Discord etc.)](#link-previews-discord-etc)
- [How the site is put together](#how-the-site-is-put-together)
- [Credits and licences](#credits-and-licences)

---

## Editing the site

Everything you'd edit by hand is in **`content/`**:

| File | What it is |
|---|---|
| `content/index.md` | Home page (welcome text, guide cards) |
| `content/_guides/*.md` | The guides, served at `/guide/<file name>/` |
| `content/_guides/minors.md` | Trans healthcare below 21, served at `/guide/gender-affirming-healthcare/below-21/` |
| `content/_side/*.md` | Side pages linked from a guide but not listed in menus (e.g. catchment areas) |
| `content/about.md` | `/about/` |
| `content/friends.md` | `/friends/` (the page itself; its listings are in `_data`) |
| `content/guide.md` | `/guide/`, the list of all guides |
| `content/copyright.md`, `content/disclaimer.md` | Linked from the footer |
| `content/404.md` | Not-found page |
| `content/_data/friends.yml` | Organisations on `/friends/` |
| `content/_data/helplines.yml` | Crisis helplines at the top of `/friends/` |
| `content/_data/notices.yml` | Wording of the distribution notice at the top of each guide |

Pages are Markdown, with some HTML for things Markdown can't do (callouts, cards, tables).
Copy the structure of an existing page when adding something similar.

**Links between pages:** write them as `{{site.baseurl}}/guide/haircuts/` so they work
wherever the site is hosted. Link to a section by its heading id, e.g.
`{{site.baseurl}}/guide/haircuts/#h-…`. Headings converted from Notion have their id in
the `{: #h-…}` line underneath; headings you add get one automatically from their text
(e.g. `## Useful links` becomes `#useful-links`), or set your own with a `{: #my-id}` line.

**Images:** put them in `assets/img/` (or alongside a guide's existing ones in
`assets/notion/<guide>/`) and reference them from the site root, e.g.
`/assets/img/example.png`.

## Page settings (front matter)

The block between `---` lines at the top of each page:

| Setting | What it does |
|---|---|
| `title` | Page title (shown lowercased as the heading; used as-is in tabs, cards and previous/next links) |
| `nav_title` | Short name in the `/guide` menu |
| `order` | Position in the menu, the home page cards and previous/next links (decimals are fine, e.g. `1.5`) |
| `nav_group` | Which menu group (1, 2, 3…); a dashed line separates groups |
| `parent` | Makes the page a subpage of that guide: adds a "← back" link and leaves it out of previous/next |
| `permalink` | Custom URL (otherwise `/guide/<file name>/`) |
| `date`, `last_modified_at` | "Published … · Updated …" under the title |
| `cover`, `cover_position` | Banner image and its vertical focal point (`"70%"` shows more of the bottom) |
| `cover_full: true` | Show the whole cover image instead of a cropped banner |
| `cover_alt` | Description of the cover for screen readers |
| `label` | `transmasculine` (blue) or `non-binary` (purple) accent colours |
| `image`, `description` | Override the link-preview image or text (see below) |
| `converter: keep` | Tells the Notion converter never to overwrite this page (see below) |

## Running it locally

You need Ruby (3.2 or newer) and Bundler.

```bash
bundle install
```

```bash
bundle exec jekyll serve
```

Then open http://localhost:4000. Most changes appear on refresh; changes to
`_config.yml` or `_plugins/` need the server restarted.

## Deploying

`.github/workflows/jekyll.yml` builds the site with Ruby 3.4 and publishes it to GitHub
Pages whenever `main` is pushed (or when run by hand from the Actions tab). In the
repository settings, **Pages → Source** must be set to **GitHub Actions**.

`Gemfile.lock` lists both macOS and Linux platforms, so the same lockfile works locally
and on GitHub. If you update gems on a Mac, keep Linux in it:

```bash
bundle lock --add-platform x86_64-linux
```

## Bringing in pages from Notion

`_tools/notion2jekyll.py` converts Notion HTML exports (zips in `_sources/`, which is not
committed) into pages. Every existing page is marked `converter: keep`, so the converter
**never overwrites** them; edit pages directly instead.

When a newer export of an existing page is added to `_sources/`, the converter compares it
with the last Notion version it saw (`_tools/notion-sync/baseline/`) and, if Notion
changed, saves the new version and a diff of just the Notion edits to
`_tools/notion-sync/incoming/`, to merge into the hand-edited page by hand. After merging:

```bash
python3 _tools/notion2jekyll.py --accept <page file name>
```

A brand-new page is converted straight into `content/` once its Notion page id is added
to `PAGES` in the converter. It needs `beautifulsoup4` (`pip install beautifulsoup4`).
Conversion notes are written to `_tools/conversion-report.md`.

## Link previews (Discord etc.)

`_plugins/social.rb` fills in each page's preview image and description when the site is
built:

- **Image:** guides with a cover use a branded preview from `assets/img/previews/` (the
  cover with the overlay `_tools/assets/preview-overlay.png` in the top-right corner).
  Pages without a cover use `assets/img/social-card.png`.
- **Description:** the page's first real paragraph (skipping the distribution notice,
  tables of contents and tables); the home page uses the site description in
  `_config.yml`.
- **Colour:** the embed's colour bar follows the page's accent (pink, or blue/purple for
  labelled guides).

Set `image:` or `description:` in a page's front matter to override either. After
changing a cover or the overlay, regenerate the previews (needs ImageMagick):

```bash
python3 _tools/make_previews.py
```

## How the site is put together

| Path | What's there |
|---|---|
| `_config.yml` | Site settings (title, address, description, collections, `noindex`) |
| `_layouts/` | Page templates: `default` (shell), `guide` (title, cover, dates, previous/next), `page` |
| `_includes/` | Reusable pieces: header and `/guide` menu, footer, home cover, guide cards, previous/next links, notices, `/friends` lists |
| `_sass/` | Styles: `_colors.scss` (brand palette, light/dark themes), `base.scss` (layout, type, header), `_notion.scss` (callouts, tables, cards, images), `_home-cover.scss`, `_friends.scss` |
| `assets/js/` | Small scripts: theme and text-size toggles, menu, image zoom (`nav.js`, `prefs.js`), database filters, table layout, relative dates, home cover stars |
| `assets/img/` | Logos (`brand/`), covers, link previews (`previews/`), home cover artwork |
| `assets/notion/` | Images from the Notion exports, by guide |
| `assets/fonts/` | Apfel Grotezk (logo font), with its licence |
| `_plugins/social.rb` | Link-preview images and descriptions |
| `_tools/` | Notion converter and preview generator (not part of the built site) |

**Brand:** colours are the Transcend (2023) palette (`_sass/_colors.scss`); type is IBM Plex
Mono (body), Inter (headings) and Apfel Grotezk (logo). The light/dark toggle and three text
sizes are remembered per browser.

## Credits and licences

- Site content: the Transcend Library and its contributors (see `/copyright/`).
- Transcend logo and wordmark: © 2023 Naaliyah Noor Daneen, under the Transcend Rebrand
  Asset Package licence.
- Theme: [jekyll-theme-console](https://github.com/b2a3e8/jekyll-theme-console) (MIT).
- Fonts: Apfel Grotezk, IBM Plex Mono and Inter (SIL Open Font Licence).
- Home page cover adapted from three CodePens (MIT):
  [Morning Sun Gradient](https://codepen.io/niallains/pen/RwjNPjM) by Niall,
  [Night Sky with twinkling stars](https://codepen.io/getdesigning/pen/xbGjJX) by
  @getdesigning, and [Clouds over Springfield](https://codepen.io/z4p0/pen/ExmjMJx) by @z4p0.
