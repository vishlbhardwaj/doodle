# doodle

A skill for coding agents (Claude Code, Codex, Cursor and others that read `SKILL.md`)
that draws anything as a code-generated line illustration: objects, products, people,
plants, buildings, landscapes, scenes and storyboards. Every drawing is authored as
geometry and rendered by a seeded "hand", so the result looks drawn. It is judged as a
finished line drawing first, with colour as a second pass. The engine is plain HTML and
JS, bundled inside the skill, with nothing to install.

```sh
npx skills add vishlbhardwaj/doodle
```

Then ask your agent to "doodle a bicycle" or "draw a rainy street in pen and ink".
Each drawing is saved as two images in `doodles/<name>/` in your project, one in
colour and one as line only, and the folder opens when it is done. The engine sits
in a hidden `.doodle/` folder next to it.

## Better results: give it a reference

Attach a photo or image of what you want and say what to take from it:
"doodle this chair", "draw my desk from this photo", "a street like this one, at night".
The agent measures the reference (proportions, the shapes that make the thing
recognisable, its palette) and builds the drawing from those numbers instead of from
memory. That is the difference between "a chair" and *this* chair.

A good reference shows the subject clearly from the angle you want, with nothing
covering it. A product photo, your own snapshot or a rough sketch all work. Without one,
the agent writes the proportions down first and works from those.

A doodle can also be **registered** to its photo: same frame, same size, every edge
where the photo's edge is, so the two can be swapped or wiped between in a video
(`references/registration.md`).

## What's inside

| | |
|---|---|
| `SKILL.md` | the contract the agent follows: forms, classes, hard rules, routing |
| `references/` | the detail per topic (line-first, products, people, plants, landscapes, registration…) |
| `engine/` | the drawing engine: `src/` modules, `bin/` scripts (`new.sh`, `render.sh`, `check.sh`, `shot.sh`), templates |

## Claude Code plugin

Or install it as a Claude Code plugin:

```
/plugin marketplace add vishlbhardwaj/doodle
/plugin install doodle@vishlbhardwaj
```

## Requirements

Chrome or Chromium (for rendering and checks; set `CHROME=/path` if it is somewhere
unusual) and python3.

## Licence

MIT, see `LICENSE`.
