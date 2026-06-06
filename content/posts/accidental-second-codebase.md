+++
title = "We Accidentally Built a Second Codebase"
date = '2026-06-01T20:42:30+01:00'
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Akshay Katyal"
authorTwitter = "mrdhat" #do not include @
cover = ""
tags = ["ai", "engineering", "claude-code", "tech-debt"]
keywords = ["ai", "engineering", "claude", "skills", "tech-debt", "platform"]
description = ""
showFullContent = false
readingTime = true
hideComments = true
draft = false
+++

A few weeks ago, I deleted 93 skills in a single pull request.

If you haven't worked with them, a skill is a small bundle of instructions you hand to the model, a workflow you've written down once so it runs the same way every time, the kind of thing you'd reach for when you're chasing down a flaky test, or pulling together the context for an incident at 2 am. Teams bundle them into plugins and publish those to a shared marketplace, and any engineer can install whichever plugins they want.

I went looking because people had started pointing out the obvious problem: there was no quality control on any of it. Anyone could publish anything, and given enough time, anyone could. Nobody had actually decided that should be the rule; it just became the rule because nobody had decided otherwise. So I opened the list one afternoon expecting to find a few duplicates and tidy them up.

My assumption going in was that most of them were load-bearing. Someone had written each one for a reason, someone depended on it, and pulling it would quietly break a workflow I'd never heard of. That assumption, more than anything, is what has let the list grow for so long. Everyone treated every skill as untouchable, me included, because you could never quite be sure who was relying on it.

Reality, as always, was less dramatic. Those 93 skills belonged to the same plugin (which had 96 skills in total), and when I checked, every one had been invoked exactly zero times in 60 days. So I pulled them, opened a PR, and merged it.

Nothing broke. I'd assumed at least one person would DM me about it, whoever had written one of the deleted skills, maybe, but nobody did. It was a little funny, to be honest. We'd been debating deleting these for a while. We kept not pressing the button, because we couldn't tell whether anyone would care, or worse, whether it would put people off writing skills altogether, and we genuinely still want people writing skills. The answer turned out to be that nobody noticed at all.

And that left me with a question I didn't have a good answer to: how had we ended up maintaining dozens of things that nobody was using?

## The one that made it click

The clearest example isn't even one of the 93 I deleted. It's one I left exactly where it was, a skill that explains to the model how to find its way around our codebase, where the important things live and how the layers fit together. It was a perfectly sensible thing to write, and I'd bet it saved people real time for a while.

Then the codebase moved, the way codebases do, and the skill stayed where it was. Now it hands the model a map to a renovated building.

There's a worse problem buried under that one. Even on the day it was written, the map was only ever accurate for the corner of the codebase the author happened to work in. Anyone working somewhere else got directions that were confident, specific and wrong. The model is perfectly capable of opening the repo and figuring out the layout for itself, but instead of letting it do that, we sat it down and told it the way things are, incorrectly.

I keep coming back to this one because of what it exposes. Nobody touched the skill. Nobody edited a mistake into it. It went bad while sitting perfectly still, because the thing it described kept moving, even though it didn't. And honestly, the original sin was writing it down at all: we took something the model could work out on its own, froze one person's snapshot of it, and signed ourselves up to maintain that snapshot forever. A map you have to keep redrawing is worse than letting the model read the territory.

Someone should look at that skill hard and probably kill it, but it's still sitting there doing its thing. The 93 I deleted were the easy case; nobody used them, so nothing pushed back when they vanished. The genuinely awkward ones are skills like this, the ones still in use, because being used is what keeps them safe from scrutiny and also what makes them a liability.

## A different kind of debt

Normal technical debt piles up because the software keeps changing. You ship, you patch, you bolt another thing onto the side, and the accumulated weight of all those changes is the debt.

A lot of what I was looking at worked the other way around. The skill itself never changed; everything around it did. The model would improve, so a workaround, a skill had carefully spelt out, wasn't needed anymore. Or the tooling improved, and the manual steps it walked you through got handled elsewhere. Or the codebase moved, and the skill kept pointing to where things used to be. Or a team got reorganised out of existence, and its skills quietly outlived it.

This is much harder to catch than the ordinary kind, because nothing in your diff history points at it. The file looks completely fine; `git blame` tells you nothing useful, and the only way actually to find the rot is to know what's changed _outside_ the file. Almost nobody is doing that against a list of skills they've half forgotten they own.

And it's everywhere once you start looking. Roughly half the skills in our catalogue have a single commit to their name, written once and never touched again, and most have had only one author. Write a skill once, never open it again, and it just carries on describing a world that has quietly moved on without it.

## Every one of them made sense

There's no villain in any of this, by the way. I wasn't staring at a pile of bad decisions.

The pattern will be familiar to anyone who's shipped software. A team notices they keep doing the same dance over and over, so they write it down as a skill and stop doing it by hand. It works well enough that they share it. The next team sees that and does the same for their own workflow. Someone gets ambitious and wires a handful of them together into an orchestration that runs end to end, and someone else adds an investigation flow for the kind of problem their team runs into every other week.

Every one of those moves is the right call at the moment it's made. Zoom in on any single decision, and it's completely rational.

Then you look up one day and the shared marketplace has more than 600 skills, spread across nearly 80 plugins, any of which an engineer can install in a second. I deleted 93 and barely made a dent. Nobody set out to build this. It's just what hundreds of small, reasonable, local decisions add up to.

And none of this is new. We've all watched this story play out before; it's just that this time, it's .md files, not .rb or .py. Internal tools, one person wrote on a Friday, that three teams now can't work without. Dashboards nobody can explain but everybody trusts. Microservices that made sense as a split at the time and now mostly just exist. CI jobs that have been green so long no one remembers what they actually check. Docs that were accurate two reorgs ago. Same failure mode as ever, just moved up a layer in the stack, and we show up with the same instincts that let it grow last time: keep it, don't touch it, someone out there probably needs it.

## I stopped thinking of them as documentation

For a long time, I'd filed skills under "documentation" in my head. Helpful text, the kind of thing you write once and forget about.

That stopped feeling right, because they'd started behaving like a codebase. They change how the model acts depending on which ones are loaded, they interact with each other in ways nobody intended, and they lean on tools and on each other and on half-stated assumptions about the world they run in. And like any other code, they go stale and need looking after, and once in a while one of them needs deleting outright.

Don't get me wrong, I'm not trying to push the analogy too far. They don't compile and the syntax doesn't matter one bit. But the maintenance burden is real, and on that score they have far more in common with a codebase than with a page on the wiki. We'd quietly grown a second codebase, written mostly in English, and nobody was treating it like one.

I wasn't the only one landing there. Months before I deleted anything, an engineering leader had said much the same thing in a Slack thread I only came across later:

> Our skills etc. are now our "code". Have we discussed what a quality ensuring pipeline would look like here?

## Most skills should probably die

If there's one thing I have started to believe, it's that most skills should eventually be deleted, and that this is a sign of health rather than failure.

Think about what a skill usually is the day it gets written:

- a workaround for something the model couldn't do well yet
- a nudge, to push its behaviour in some direction you wanted
- a stand-in for a capability that didn't exist at the time
- a bit of knowledge that, until then, had only lived in one person's head

Now give it a year or two. The model gets better at the very thing some workaround was working around, the tooling quietly absorbs the manual steps, the product moves on, and the reasons a skill existed start expiring one by one without anyone noticing. By then the workaround isn't needed, the capability has properly shipped, and whatever knowledge the skill held has been written down somewhere more durable. The skill has done its job. The honest thing to do is retire it.

Which is why a catalog that only ever grows isn't the good sign it looks like. It usually means nothing is being allowed to finish its job and leave.

## What I'd actually want to argue about

I'm not going to pretend I have this figured out. Internally I've floated a few things, a quality bar in front of the catalog, actual named owners, some way of retiring skills on a schedule, and I genuinely don't know how many of them are any good. The honest situation is that nobody has worked out yet what "good" even looks like here.

Is it benchmarks for skills? Quality metrics? Alerting that fires when a skill's assumptions have gone stale? I really don't know. The one thing I'm fairly confident about is that we shouldn't end up settling this the same way we built the catalog in the first place, by default, one reasonable little addition at a time, until you glance up and it's six hundred deep. This is the sort of thing worth deciding on purpose, for once.

So these are the questions I'd want a room full of engineers to actually fight about:

- What do you measure, when "barely used" and "rarely needed but critical" look identical from the outside?
- What earns deletion, and who gets to pull the trigger?
- How much governance can you add before you kill the thing that made skills useful in the first place, that anyone could write one?

The deletion was the easy part. I still don't know how you keep hundreds of these honest as the ground keeps shifting under them, and I don't think anyone else does yet either.
