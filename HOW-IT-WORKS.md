# How my Hugo bio site works

A small personal website about me, built with Hugo: a dark blue and green colour scheme, and one screenshot for each project. This file explains every part so I can present it and answer questions.

The project follows *Static Site Generators in the Age of AI* by Dr. Polla Abdulhamid Fattah (https://polla.dev/ssg-book/). It uses the book's one-template approach (Chapters 1 to 5) and the template blocks from Chapters 9 and 10. Section 8 says exactly which parts are the book's code and which are my own.

---

## 1. The idea in one minute

Hugo is a **static site generator**. It reads plain text files (Markdown for the writing, one HTML template for the page shape, one CSS file for the look) and **builds finished HTML pages before any visitor arrives**. (Chapter 1, "1.1 A quick picture of what we are building": I write the content, Hugo builds the page, the browser displays the result.)
```
content/*.md        ─┐
layouts/all.html    ─┼──►  hugo  ──►  finished HTML pages  ──►  browser
static/css/site.css ─┤
hugo.toml           ─┘
```

"Static" means the pages are already finished when they are served. It does not mean the site never changes: I change a file and Hugo rebuilds.

There is no server code and no database.

---

## 2. The files and what each one is responsible for

This is the table from Chapter 1, "1.6 Understand what changed", filled in for this project.

| File | Responsibility | Change it when I want to… |
|---|---|---|
| `hugo.toml` | Site-wide settings: the site name (`Muhammad Yaseen`), the language, the base address | rename the site |
| `content/_index.md` | The **home page** text | change the home page |
| `content/about/index.md` | The About page: what defines my work, skills, languages | update how I describe myself |
| `content/experience/index.md` | The Experience & Education page: jobs, university, self-directed learning | update my background |
| `content/contact/index.md` | The Contact page: email, profile links, availability | change how people reach me |
| `content/projects/_index.md` | The **Projects section** introduction | change the intro above the project list |
| `content/projects/<name>/index.md` | **One project** each (5 of them) | add, edit or remove a project |
| `content/projects/<name>/cover.webp` | That project's screenshot, **beside** its `index.md` | change a project's picture |
| `layouts/all.html` | The one HTML template used by every page | change the page structure |
| `static/css/site.css` | Colours, spacing, project cards | change the look |

**Content, layout, and style are separate files.** I can rewrite my bio without touching the HTML, and restyle the site without touching the writing. That separation is the main point of Chapter 1.

Never edit `public/` if it appears. It is generated output, and the next build overwrites it (Chapter 1, 1.6).

```
my-bio-site/
├── hugo.toml
├── content/
│   ├── _index.md                       ← home page
│   ├── about/index.md
│   ├── experience/index.md
│   ├── contact/index.md
│   └── projects/
│       ├── _index.md                   ← the Projects section page
│       ├── department-lectures-platform/             ← index.md + cover.webp
│       ├── kib-inventory-sales-system/
│       ├── kurdistan-journalists-syndicate-analytics/
│       ├── loan-management-system/
│       └── university-feedback-system/
├── layouts/all.html
└── static/css/site.css
```

---

## 3. How to run it

Open a terminal in the `my-bio-site` folder (the one that contains `hugo.toml`).

**Preview while I work:**

```bash
hugo server
```

Then open http://localhost:1313/. Hugo rebuilds and refreshes the browser every time I save a file. Stop it with `Ctrl+C`. (Chapter 1, "1.4 Open your first preview".) Only my computer can open this address.

**The book's standard check:**

```bash
hugo --minify --panicOnWarning
```

This builds the whole site and turns any warning into an error. If it finishes with no error, the templates and front matter are valid. It does **not** prove that the writing is true or that every link is the one I meant; the book repeats this in every chapter. This command also creates a `public/` folder, which is generated output.

Hugo version used: 0.167.0 extended. The book needs 0.146.0 or later.

---

## 4. `index.md` versus `_index.md`

This is the question most likely to be asked. The two names are **not interchangeable** (Chapter 3, "3.4 Make Articles a useful landing page", the box "Keep the two index filenames distinct").

| File | Meaning | Address it makes |
|---|---|---|
| `content/_index.md` | The home page | `/` |
| `content/projects/_index.md` | The text of the **Projects section** (a page that holds other pages) | `/projects/` |
| `content/projects/loan-management-system/index.md` | **One single page** (a "leaf bundle") | `/projects/loan-management-system/` |
| `content/about/index.md` (and `experience`, `contact`) | One single page each | `/about/`, `/experience/`, `/contact/` |

- Underscore (`_index.md`): a page that **contains other pages**, such as a section or the home page.
- No underscore (`index.md`): a page that stands alone. Its folder can hold the page's images too (Chapter 2, "2.3 Understand the file you just created").

**The folder name becomes the address.** I can change a project's `title` freely; if I rename its folder, its address changes and links to it break (Chapter 3, 3.7).

---

## 5. Front matter

Every content file starts with a block between two `---` lines. It is **front matter**, in YAML (Chapter 1, 1.5). Below it is the Markdown writing.

A project file looks like this:

```yaml
---
title: "Loan Management System"
description: "A loan platform for financial institutions with approval workflows and admin dashboards."
params:
  tools:
    - "Laravel"
    - "PHP"
    - "Filament"
    - "MySQL"
    - "JavaScript"
    - "Chart.js"
---

## What it is

A loan management platform for financial institutions. It includes approval workflows and admin dashboards.

## Screenshot

![A screenshot of the Loan Management System](cover.webp)

[Back to Projects](../)
```

| Part | Kind of value | Used by the template as |
|---|---|---|
| `title` | text | `.Title` |
| `description` | text | `.Description` |
| `params:` → `tools:` | a **list** under a custom key | `.Params.tools` |
| `![alt text](cover.webp)` in the body | an image file beside `index.md`, with alt text | an `<img>` on the page (Chapter 2, "2.6 Illustrate the article with your own screenshot") |
| everything after the second `---` | Markdown | `.Content` |

`title`, `description` and `draft` are names Hugo already knows. My own information (the tools list) goes under `params`, which is Hugo's convention for custom values (Chapter 9, "9.3 Read only the YAML you need"). Indentation uses spaces, never tabs.

Body headings start at `##` because the layout already prints the title as the page's one `<h1>` (Chapter 2, 2.2).

Links between pages are **relative** and never start with `/`. For example `[Back to Projects](../)` and `[About](about/)`.

---

## 6. Walk through the template, line by line

`layouts/all.html` is the only template. Hugo uses it for the home page, About, Experience, Contact, the Projects page, and every project page. `{{ ... }}` is where Hugo fills something in; everything else is ordinary HTML.

### 6.1 Which template does Hugo pick?

There is no other layout, so Hugo falls back to `layouts/all.html` for everything. Chapter 11 describes it as the layout used "when no more specific layout applies". Each page is the same shell with different content poured in.

### 6.2 The shell (Chapter 1 starter, Chapter 3 nav, Chapter 4 footer)

| Part | What it does |
|---|---|
| `<title>{{ .Title }} \| {{ .Site.Title }}</title>` | Browser tab text: the page title, then the site title from `hugo.toml` |
| `<link rel="stylesheet" href="{{ "css/site.css" \| relURL }}">` | Loads `static/css/site.css`. `relURL` builds the link relative to the site's base address |
| `<a class="skip-link" href="#main">` | A keyboard shortcut past the menu to the content (Chapter 4) |
| `<p class="site-name">{{ .Site.Title }}</p>` | Shows `Muhammad Yaseen` on every page |
| `<nav>` with five `relURL` links | Home, About, Experience, Projects, Contact (Chapter 3, "3.6 Add navigation and clear starting points") |
| `<h1>{{ .Title }}</h1>` | The page heading from front matter |
| `{{ .Content }}` | The Markdown body converted to HTML |
| `<footer>` | Two paragraphs; the second has `class="footer-note"` and links to About (Chapter 4, "4.3 Make one persistent HTML change") |

### 6.3 The dot (`.`)

In a Hugo template the dot means **"the thing I am currently looking at."** It starts as **the page being built** and changes inside `with` and `range`. Then it returns to the page after the matching `{{ end }}`. This is the idea to master in Chapter 10.

So `{{ .Title }}` means "the title of the current page".

### 6.4 `with .Description` (Chapter 10, "10.3 Use `with` when a value is optional")

```html
{{ with .Description }}
  <p class="lead">{{ . }}</p>
{{ end }}
```

| Position | What `.` is |
|---|---|
| Before `with` | the page |
| Inside `with` | the description **text** |
| After `end` | the page again |

`with` runs its block **only if the value exists**. The home page and About have no `description`, so they get no empty paragraph. Project pages, Experience and Contact do, so they show it.

### 6.5 The tools list: `with` + `range` (Chapter 10, "10.4 Repeat markup for the tools list")

```html
{{ with .Params.tools }}
  <ul class="tools">
    {{ range . }}
      <li>{{ . }}</li>
    {{ end }}
  </ul>
{{ end }}
```

| Position | What `.` is |
|---|---|
| Before `with` | the page |
| Inside `with`, before `range` | the **tools list** |
| Inside `range` | **one tool name** (`Laravel`, then `PHP`, …) |
| After the inner `end` | the tools list |
| After the outer `end` | the page |

`range` repeats its block once per item in a list. It prints one `<li>` per tool, so I never write a fixed number of list items. Only project pages have `tools`, so only they show this block. The `class="tools"` only lets the CSS draw the tools as small green tags.

### 6.6 The project list (Chapter 10, "10.5 Generate the Projects list from its pages")

```html
{{ if and .IsSection (eq .Section "projects") }}
  <h2>Selected projects</h2>
  <ul class="projects">
    {{ range .RegularPages.ByTitle }}
      <li>
        <a href="{{ .RelPermalink }}">
          {{ with .Resources.GetMatch "cover.webp" }}
            <img src="{{ .RelPermalink }}" alt="">
          {{ end }}
          {{ .Title }}
        </a>
        {{ with .Description }}
          <p>{{ . }}</p>
        {{ end }}
      </li>
    {{ else }}
      <li>No projects to display yet.</li>
    {{ end }}
  </ul>
{{ end }}
```

Read it in order:

1. `if and .IsSection (eq .Section "projects")`: only continue if this page is a **section page** and its section is `projects`. That is the `/projects/` page and nothing else.
2. `.RegularPages`: the single pages inside this section, meaning the five project `index.md` files.
3. `.ByTitle`: sort them alphabetically by title. The order of the folders on disk does not matter.
4. `range`: loop over them. **Inside the loop the dot is one project page**, so `.Title` is the project's name and `.Description` is its description.
5. `.RelPermalink`: Hugo gives me each project's address. I never type it by hand.
6. `else`: what to show if there are no projects.

**The one line that is not in the book** is the image: `{{ with .Resources.GetMatch "cover.webp" }}`. A project's folder holds `index.md` and `cover.webp` together, and Hugo calls the extra files in that folder the page's **resources**. `GetMatch "cover.webp"` finds the file with that name in *this project's own folder*. Inside the `with`, the dot is that image file, so `.RelPermalink` is the image's address; after `end` the dot is the project again, which is why `{{ .Title }}` works. Because it is a `with`, a project with no `cover.webp` just shows no picture.

`alt=""` is deliberate here: the picture is only decoration next to the project's name, which is already the link text.

**The list builds itself from the content.** There is no hand-written list of projects anywhere.

---

## 7. The look: colours and the CSS

The styling is in one file, `static/css/site.css`. It starts from the book's Chapter 1 stylesheet and changes the colours. The colour scheme is a dark blue background with a green accent, as on my portfolio.

| Colour | Where it is used |
|---|---|
| `#0a192f` dark navy | page background, project cards |
| `#112240` lighter navy | the main content box |
| `#233554` | thin borders |
| `#ccd6f6` pale blue | body text |
| `#e6f1ff` almost white | headings |
| `#8892b0` grey-blue | quieter text: footer, intro line, card descriptions |
| `#64ffda` mint green | links, site name, tool tags |

To change a colour, search for its code in `site.css` and replace it everywhere it appears.

What each part of the CSS does:

- **Selector and rule** (Chapter 5, "5.1 Give the footer note a style"): `.tools li { ... }` means "every `<li>` inside an element with `class="tools"`". The properties inside the braces are `property: value;` pairs.
- **`main { background: #112240; border-radius: ... }`**: the rounded content box.
- **`.tools`**: `display: flex; flex-wrap: wrap` puts the tool tags in a row that wraps onto the next line, and `border-radius: 999px` makes them pill shaped.
- **`.projects`**: `display: grid; gap: 1rem` stacks the project cards with space between them. Each card is one `<li>` with a border and a rounded corner.
- **`.projects img`**: `aspect-ratio: 16 / 9; object-fit: cover; object-position: top` makes every screenshot the same shape, cropped from the top, so the cards line up.
- **`article img`**: the book's own rule from Chapter 2, "2.6", so an image never overflows its box.
- **`width: min(100% - 2rem, 48rem)`**: the page is at most `48rem` wide and keeps a margin on narrow screens (Chapter 5, "5.6 Understand the responsive behaviour already supplied").
- **Hover effects:** `a:hover` makes a link brighter with a thicker underline, and `.projects li:hover` gives a project card a green border and lifts it 3 px. `transition: ... 0.2s` makes the change smooth instead of instant.
- **`a:focus-visible` and the skip link**: kept from the book so keyboard users can see where they are.

---

## 8. Which code comes from the book and which is mine

| Piece | Source |
|---|---|
| `hugo.toml` | Chapter 1 starter file, "File 1". **Only `title` is mine.** |
| `layouts/all.html`: the document shell, skip link, `<main id="main">`, `<article>`, `<h1>`, `{{ .Content }}` | Chapter 1 starter, "File 3". |
| The `<nav>` | Chapter 3, 3.6 pattern: five links with `relURL`. **Mine:** the destinations (Home, About, Experience, Projects, Contact). |
| The footer | Chapter 4, 4.3 shape with the `footer-note` class. **The sentences are mine.** |
| `{{ with .Description }}` | Chapter 10, 10.3. **Mine:** the `<p>` has `class="lead"`. |
| The tools list | Chapter 10, 10.4. **Mine:** a CSS class, and no "Tools:" label line. |
| The project list | Chapter 10, 10.5: the condition, `range`, `.RegularPages.ByTitle`, `.RelPermalink`, `with .Description` and `else` are the book's code. **Mine:** the `<ul class="projects">` class and the optional image (`.Resources.GetMatch`). |
| `static/css/site.css` | The Chapter 1 starter stylesheet and the Chapter 5 `.footer-note` rule, with **my colours**. The `article img` rule is Chapter 2, 2.6. **Mine:** the colours, the `.lead`, `.tools` and `.projects` rules. |
| Project front matter | The shape from Chapter 9, 9.2 and 9.3 (`title`, `description`, `params.tools`). |
| The screenshots (`cover.webp`) | Copied from my portfolio. Putting an image beside `index.md` follows Chapter 2, 2.6. |
| All the writing | Mine, taken from the pages of my portfolio at https://muhammad-yaseen.dev/ (home, About, Experience, Projects, Contact, Resume). The wording of the project descriptions and the job bullets is copied from there. |

The book's project model also has a `status` field (Chapter 9). I left it out: my portfolio does not give a progress state for each project.

---

## 9. What I deliberately left out, and why

The lecturer asked for a simple project, so I stopped at what is easy to explain.

| Left out | Why |
|---|---|
| A theme | The book starts without one (Chapter 1). Everything on screen is files I can read. |
| `baseof.html` and partials | That refactor is Chapter 11. |
| A home-page hero, skill logos, special fonts, animations | They would add template and CSS ideas that are not in the book. The site uses the normal system font. |
| A profile photo | My portfolio's home and About pages have none, so there is none here. |
| A JSON data file, search, a contact form, JavaScript | Chapters 12, 15, 17. The portfolio's message form needs a server, so the Contact page lists my email and links instead. |
| The portfolio's animated counters ("0+ years building", and so on), the Kurdish toggle and the resume page | The counters show placeholder zeros in the page source, so I did not copy any numbers. The resume is linked as a PDF on my portfolio. |
| Kurdish translation | A translation should be reviewed by a person who reads the language (Chapter 16, 16.6). This site is English only. |
| Git history, GitHub Pages | Chapters 6 and 7. `baseURL` is still the book's placeholder `https://example.org/`, so this is **not published**. To publish, follow `HOSTING.md` (Chapter 7, 7.3 sets the real address). |

Hugo also builds empty `/tags/` and `/categories/` pages by default. Nothing links to them. Chapter 3, 3.7 explains tags and categories without using them.

---

## 10. Live demo: add a project, do not touch the template

1. Create the folder `content/projects/my-new-project/` and a file `index.md` inside it:

   ```yaml
   ---
   title: "My New Project"
   description: "One sentence about it."
   params:
     tools:
       - "Hugo"
   ---

   ## What it is

   A short explanation.

   [Back to Projects](../)
   ```

2. Save, with `hugo server` running. The Projects page now shows **six** entries, and `/projects/my-new-project/` exists. It has no picture because the folder has no `cover.webp`.
3. Optional: copy any image into the folder and name it `cover.webp`, and the card gets its picture.
4. Delete the folder to go back to five.

---

## 11. Questions I should be ready for

**Why is the site called "static" if I can edit it?**
Because the pages are finished before a visitor arrives. Editing means changing the source files and rebuilding.

**Where does a page's title come from?**
The big heading comes from the `title` in that page's front matter. The name in the header comes from `hugo.toml`. They are different on purpose (Chapter 1, 1.6). On the home page the big heading is "Full-Stack Developer from Erbil, Iraq" and the header says "Muhammad Yaseen".

**Why is there only one layout?**
`all.html` is Hugo's fallback layout. With no other layout, every page uses it. Chapter 11 splits it into reusable parts.

**Why does the Projects page list itself?**
Because `layouts/all.html` has a `range` over the section's pages. A new project folder appears in the list automatically.

**Why do the `index.md` files have no underscore but `_index.md` does?**
See Section 4: `_index.md` describes a page that holds other pages; `index.md` is one standalone page.

**How does a project's picture get onto its card?**
The picture sits in the project's folder next to `index.md`. The template asks for it with `.Resources.GetMatch "cover.webp"` (Section 6.6).

**What does `hugo --minify --panicOnWarning` prove?**
That the site builds with no warnings. It does not prove the content is correct.

**Why no `/` at the start of the links in the Markdown?**
Relative links keep working if the site is published under a sub-path, such as a GitHub Pages project address (Chapter 7).

**Where did the colours and screenshots come from?**
From my own portfolio. The colours are in `site.css` (Section 7), and the screenshots are copied into each project's folder.

---

## 12. Checks

**Already run while this project was prepared (2026-11-23, Hugo 0.167.0 extended):**

- `hugo --minify --panicOnWarning` from this folder: exit status 0, 17 pages built.
- A scan of the built HTML: every internal link and image path resolves (110 checked, none broken), no `{{` is left in any page, and the Projects page lists five projects, each with its image.
- The add-a-project demo (Section 10) on a scratch copy of the final site, with no image in the new folder: the build passed and the list grew from five to six entries. The copy was deleted afterwards.
- Screenshots in headless Chrome of the Experience page and a project page at desktop width, taken after the content was added. The home, Projects and Contact pages were viewed on the earlier, shorter content; their layout has not changed since.

**Still for me to check in a real browser:**

- `hugo server`, then click every link in the navigation and open every project.
- Narrow the window to about 360 px wide and zoom to 200%: nothing should overflow (Chapter 5, "5.6").
- Press `Tab` once on a page: the "Skip to content" link should appear, and every link should show a visible outline.
- Say whether the colours look right, and which to change.
- **Every claim about me is correct.** The text comes from my portfolio and has not been independently verified. Check the dates, the job bullets (including "Shoplink SaaS", "Pelistank backend APIs" and the DIT internship in June to July 2026), the "currently learning Rust and Axum" lines, the project descriptions ("serving thousands of users", "over 20,000 active users", "FIFO batch costing", "regulatory compliance reporting") and the "responds within 24 hours" line. Also check the dates (Technobase intern December 2025 to February 2026, part-time February 2026 to present; university 2022 to present, fourth stage), the skills, and the contact links. The Backend, Frontend and Tools grouping of the skills is my own sorting.
- **The screenshots** show real data from live systems. They are the same images as on my portfolio, but I should decide that I am happy to show them here too.
