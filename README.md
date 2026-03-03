# Data Diversity Lab — Interactive Website

## About

This website was built for a visit to the lab in March 2026. The goal is to give a visitor a clear, engaging description of the work happening in the Data Diversity Lab. I would particularly like to show who is currently working here, what we are working on, and how it all connects. The second intention is to not openin a slide deck, which would change the structure of the meeting. 

The emphasis is entirely on interactivity. Rather than a static presentation, this site is designed to guide a live conversation. You can click through students, zoom into a network of connections, or land on a specific project depending on where the discussion goes. Nothing is locked into a linear order but more like a guided walk through our work, which will also align with the folks who work on particular topics and will be talking about their work.

---

## Structure

The site is a single `.qmd` file that renders to a self-contained `.html` file. There is only one file, no server, no dependencies to install beyond Quarto itself. Once you have rendered the document, open it fullscreen in a browser and you will be ready to go!

The file is organized into three main layers.

### YAML front matter

At the top of the `.qmd`, the YAML block tells Quarto how to render the output:

```yaml
format:
  html:
    self-contained: true
    page-layout: custom
    toc: false
    theme: none
```

`self-contained: true` combines everything including fonts, scripts, and styles, into a single `.html` file. `page-layout: custom` and `theme: none` strip out all of Quarto's default structure so the design is entirely my own.

External libraries are loaded in the `include-in-header` block: the Google Fonts (ugh) for typography and D3.js (v7) for the network visualization.

### HTML and CSS — the `{=html}` block

The entire interface is written inside a single ```` ```{=html} ```` block, which Quarto passes through directly to the output without modification.

The CSS uses custom properties (CSS variables) defined on `:root` to keep the color palette consistent across every component. Changing `--sage` or `--cream`, for instance, propagates everywhere. Animations are included such that there is a fade-in transition on tab switch, the card entrance stagger, the hover lift. All of these are CSS keyframe animations, a structure that ensures interactions are effectively lightweight and smooth.

The layout is four tabs: **Overview**, **People**, **Projects**, and **Network**. This can be expanded or reduced as needed in accordance to the goals of the presentation etc. Only the active section is displayed. Specifically, switching between tabs triggers a `fadeIn` animation and, in the case of the Network tab, initialises the D3 force simulation on first visit.

### JavaScript is used for data and rendering

All of the logic lives in a `<script>` block at the bottom of the HTML. The code is split into two parts.

**The data tables** sit at the very top of the script and are the only thing that ever needs to be edited. Two plain JavaScript arrays (`MEMBERS` and `PROJECTS`) hold all the information about people and their research. Each entry is a straightforward object with labelled fields. There are comments in the file that walk through what each field does.

**The rendering functions** read from those arrays and build the interface dynamically. Every tab (e.g. stats on the overview, the photo grid for people, the project cards, the force network) is generated from the data at page load. This means adding a new student or project never requires touching the HTML as it only requires adding a row to the relevant array.

The network visualization uses **D3.js force simulation** directly (and not through Observable). Nodes are people and projects; edges connect each person to their projects. The force layout uses a short link distance and centring forces on both axes to keep the graph compact rather than letting clusters drift to the edges. The simulation runs on first tab visit and stays interactive as nodes can be dragged and the whole graph can be panned and zoomed.

---

## Aesthetics

The visual language follows the same structure that I use in my slides: restrained, readable, with room between elements. A few specific details...

**Typography.** Two fonts from Google Fonts — *Fraunces* (a variable serif with an italic I particularly like) for display text, headings, and names. *Figtree* (a clean geometric sans-serif) is used for body text, labels, and UI elements.

**Color.** The palette is defined in CSS variables: sage green as the primary accent, cream and off-white for backgrounds, a deep forest ink for text, and gold as a secondary accent for highlighted items. Research area colors are assigned automatically from a curated set of muted tones so every field gets a distinct color.

**Space.** There is a lot of space in between elements. For instance, cards do not fill their containers. Simiarly, sections have generous top and bottom padding. The goal is a site that feels calm and considered rather than dense and packed.

**Animation.** Tab transitions fade and slide in. Stat blocks stagger their entrance. Cards lift on hover. The network nodes respond to drag with immediate feedback. All of it is CSS or D3 — no animation libraries.

**Highlighting.** Both students and projects have a `highlighted: true/false` flag. Setting it to `true` adds a colored badge to that person's card or a gold border to that project. This is useful for a visit where some work will get to be featured intentionally. Similarly, only a fraction of the students will be speaking.

---

## How to modify

### Rendering the site

```bash
quarto render lab-showcase.qmd
```

This produces `lab-showcase.html` in the same folder. Open it in any browser.

### Updating lab name or university

At the top of the HTML block, find the `<nav>` element and update the two text strings:

```html
<span class="nav-lab">The Data Diversity Lab</span>
<span class="nav-uni">· University of Arizona</span>
```

Update the overview section tagline and description in the `<section id="overview">` block similarly.

### Adding or editing a student

Find the `MEMBERS` array in the script. Each person is one object in the `.qmd`. Copy any existing block and fill in the fields:

```js
{
  name:        "First Last",
  role:        "PhD Student",
  photo:       "firstname.jpg",   // place this file next to the .qmd
  projects:    ["p1", "p3"],      // must match ids in PROJECTS below
  keywords:    ["Topic A", "Topic B"],
  area:        "Field X",         // controls color; must match a project area
  highlighted: false
},
```

Leave `photo: ""` and the site will show styled initials instead. The photo file needs to sit in the same folder as the `.qmd` file. You can also use links to images.

Alumni are counted separately in the Overview stats. To mark someone as an alumnus, include the word `"Alumni"` anywhere in their `role` string:

```js
role: "Alumni · Now at [Organisation]",
```

### Adding or editing a project

Find the `PROJECTS` array. Each project is one object:

```js
{
  id:          "p5",                   // unique and no spaces
  name:        "Project Name",
  description: "One short paragraph describing the work.",
  keywords:    ["Topic A", "Topic B"],
  area:        "Field X",
  highlighted: false
},
```

Then add `"p5"` to the `projects: []` array of every member who works on it.

### Adding a new research area

Note that there is no configuration needed. Just use a new string in `area:` for a member or project (e.g `"Field W"`) and the site assigns it a color automatically, adds it to the filter buttons, and includes it in the network legend.

### Highlighting for the visit

To flag a student who will speak, or a project that will get dedicated time, set:

```js
highlighted: true
```

This adds a colored **★ Featured** badge to their card (for people) or a gold border with a **★ Highlighted** label (for projects). Set it back to `false` for a different event.

### Changing the overview quote or description

In the HTML, find the `<div class="overview-intro">` block and edit the text directly:

```html
<div class="overview-intro-title">"Your quote here."</div>
<p>Your description here.</p>
```

### Adjusting colours

All colors are CSS variables at the top of the `<style>` block. Change the values there and they propagate everywhere:

```css
:root {
  --sage:  #7aab85;   /* primary accent */
  --gold:  #c8a84b;   /* highlight colour */
  --cream: #f8f5ef;   /* page background */
  --ink:   #1c2b22;   /* primary text */
}
```

Research area colors are drawn from the `AREA_PALETTE` array in the script. Add or reorder hex values there to change which colors get assigned to which areas.
