+++
title = "Notes on notes"
description = "How to get thoughts out quickly."
date = 2025-08-30 18:00:00
tags = ["notes"]
+++

```markdown
# How to format notes for getting thoughts out quickly
- Start with a Markdown file.
- Each top-level section identifies a subject.
	- Typically, only level 1 headings are used.
- Within each section, each thought is an item in an unordered list.
	- Immediately related thoughts are just sub-items.
- These can nest as deeply as what looks reasonable, but it's sometimes better
  to just jump back to the root, and let the next thought build off of the
  previous.
- don't even bother with proper syntax. it doesn't add anything
	- If you do, by chance, it doesn't matter. Just leave it.
- insert thoughts wherever it feels right
- since they're just list items, thoughts can easily be shuffled around, and
  relevant thoughts can be grouped together
- the main purpose is to have these thoughts written down with just enough
  fidelity that you can rebuild the full thought when necessary
- you can always revisit thoughts and expand upon them later

Paragraphs are avoided for this because they introduce the cognitive effort of
having format them. Should this thought be a part of the current paragraph, or
should a new one be started? Or maybe this paragraph is too long, and it's hard
to pick out the important information.

That said, paragraphs are great for fully-formed thoughts. Depending on the
subject, once you've fully explored it and accumulated enough thoughts, you can
reformat them into something with more narrative, or that is at least more
readable.

- or don't. not every subject needs it

- [>]: create syntax only on the fly
- [x]: don't make a system out of it that you have to memorize
- [x]: if this syntax is making it hard to get thoughts out, then get rid of it
- [o]: if this syntax is obvious, then it is correct

# Organizing files
- My personal notes are organized roughly by subject.
	- Each directory is a subject.
	- `notes.md` usually serves as the main notes for a directory.
- Each of my projects usually has a personal `notes` directory.
	- This directory is almost always .gitignore'd or .git/info/exclude'd.
	- `notes.md` contains notes that live long-term with the project.
		- Ideally, should be helpful to me-in-six-months.
		- Parts of it might make their way into an ARCHITECTURE.md file, if
		  there's enough justification for one.
	- `TODO.md` contains temporary todo items.
		- Usually, a single list item with sub-items is enough.
		- Bigger todos can have their own sections.
		- In any case, when the todo is done, it gets removed.
		- That's it. No issue trackers. No kanban. No bells, no whistles, no
		  overhead.
		- I see this as ideal for single-person projects.
		- For multi-person projects with a tracking system, this can still be
		  useful for the individual: Quickly get the todo down now, organize it
		  later.
	- Can contain a number of other note-like things, like references, short
	  scripts, or prototypes.

# Conclusions
- notes don't have conclusions
- they do not finish
- they do not end
```
