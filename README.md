# Security Notebook

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This is where I write down what I'm learning as I go through cybersecurity — networking, Linux, SOC work, application security, cloud, all of it. It's my own notebook, kept in public.

I'm not writing this as a tutorial for anyone else. It's what I'd write for myself anyway, just done in the open instead of hidden in a private folder. If it happens to help you understand something too, that's great. If you spot something wrong or think it could be explained better, I'd genuinely like to hear it.

## Contents

- [Why I'm doing this in public](#why-im-doing-this-in-public)
- [How it's organized](#how-its-organized)
- [How I write a note](#how-i-write-a-note)
- [If you want to learn from this](#if-you-want-to-learn-from-this)
- [If you want to contribute](#if-you-want-to-contribute)
- [What's not here](#whats-not-here)
- [Author](#author)
- [License](#license)

---

## Why I'm doing this in public

Two reasons, honestly.

First, writing something down properly — in my own words, not copy-pasted from a course — is how I actually learn it. Watching a video and finishing a lab feels like progress, but it isn't the same as being able to explain the thing without looking anything up. This notebook is where that explaining happens.

Second, most people learning cybersecurity study alone and nobody ever sees the work. I'd rather have this be something real — a record that shows where I started and how my understanding actually grew, mistakes included. If someone else studying the same stuff finds a note useful, or catches an error and tells me, that makes the whole thing better for both of us.

## How it's organized

The folders follow the order I'm actually learning things in. A folder shows up when I reach that topic, not before. Right now I'm just starting Phase 1, so this is all there is:

```text
foundations/
    networking/
        osi-model.md
```

More will get added as I move through networking, then Linux, then Python, then into SOC, AppSec, and cloud — following my roadmap. This tree will keep growing, it just reflects wherever I actually am right now, nothing added ahead of time.

## How I write a note

Every note follows the same shape. The order matters — it moves from "what is this" to "can I actually explain it," which is roughly how understanding something really happens. Doesn't matter if the topic is a concept, a tool, an attack, or a protocol — this shape fits all of it:

```markdown
# Topic Name

## In short
One or two sentences. If I only had ten seconds to remind myself what this is, this is what I'd read.

## What it is
The plain explanation. Written like I'm explaining it to a friend who has never heard of it — no jargon, and if a technical word has to be used, it gets explained right there in the same line.

## Why it matters
Where this actually shows up. In an attack, in defense, in a tool I'd use, in a job. If I can't answer "so what" about a topic, I probably don't understand it well enough yet.

## How it works
The actual mechanism, step by step. If there's a sequence — a handshake, a request going somewhere, an attack chain — I walk through it in order, start to finish. This is the part that separates actually understanding something from just recognizing the name. If I can't write this section, I'm not done learning the topic yet.

## Key details to remember
The exact facts worth being able to recall instantly, no thinking required. Port numbers, commands, exact terms, formulas — whatever this specific topic needs. Written short, like a cheat sheet, not full sentences.

## Where I got confused
Whatever tripped me up while learning this. A wrong assumption, two terms I kept mixing up, a part I had to re-read a few times before it clicked. This is the most useful section for future me — it's usually the exact spot I'll get confused again if I ever forget this topic.

## How I'd say this out loud
The short, spoken version. The way I'd actually explain this to an interviewer or a teammate, no notes in front of me. If this section is hard to write, the note above isn't finished yet.
```

Not every note needs every section. A short note on a small tool might just use "in short," "what it is," and "key details." A deep topic like the OSI model or a TLS handshake will use all seven. I use whatever fits, and I'd rather write a short, correct note today than wait until I can fill in every section — I go back and expand notes as I understand more.

## If you want to learn from this

Read whatever's useful. Everything here is free to use — copy it, adapt it, reference it in your own notes. No credit needed, though it's always appreciated.

## If you want to contribute

You're welcome to. A few ways that actually help:

- **Found something wrong?** Open an issue, or just point it out. Nobody gets everything right the first time and I'd rather fix it than leave it.
- **Think a note is confusing?** Say so, and tell me which part. That feedback is worth more than it sounds.
- **Have a resource that explains something better than I did?** Drop a link in an issue, I'll take a look.
- **Want to add a note yourself?** Open a pull request. Keep it in the same format above, written in plain language, not copy-pasted from a textbook or a course. I'll review it before merging since this is still meant to read as one person's notebook, not a crowd-written wiki.

I'm not expecting a lot of contributions and that's completely fine — this repo works even if it's just me. But the door's open.

## What's not here

Actual tools and scripts I build — the port scanner, the log analyzer, the SIEM dashboard, and so on — live in their own separate repos. If a project needs background explanation, I'll link to the relevant note here instead of repeating it.

## Author

Written and maintained by Kairos. More of my work: [kairos.sahilmauryadev.com](https://kairos.sahilmauryadev.com/)

## License

Notes in this repo are shared under the MIT License — use them freely, no permission needed.
