---
layout: post
permalink: /blog/how-to-review-candidates
title: "How to review software engineering candidates"
date: "Aug 8, 2022"
---

A couple of years ago I wrote about [how to get the most out of a tech test][tt].
This was aimed at people interviewing for their next job, who were required to
complete a take-home test as part of the process.

I thought it may be useful to write the other side of this: you've been asked to
review a candidate's tech test, how should you approach this?

So without any preamble, here we go.

## 1. Set aside uninterrupted time for the review

Like a PR review, you want to do this properly. Make sure you have 30-60 minutes
of dedicated time, turn off distractions, let teammates know what you are doing.

The candidate spent several nervous hours on this. In a fair world you would
spend an equal amount of time reviewing it, but that is rarely how it shakes out
The absolute least that you, the person who potentially holds the future of their
career in your hands, can do is concentrate solely on their work for half an hour.

## 2. Take notes

Having two columns is a good idea, for things you liked and things you feel could
have been done better. And don't just write what you didn't like, add a sentence
about why. Once you have finished your review, send your notes to the hiring
manager and they can forward them to the candidate.

Feedback on interviews is so valuable, and yet so few companies think it is worth
their time. I guess they think they are too busy and important to give back to
the community that created them? idk. It is really not hard, and if you are a
Senior-or-above engineer, you should be able to do this in your sleep.

_(Pro tip: If you are a senior engineer who does not know how to teach and coach, then
**you are not actually a senior engineer**.)_

## 3. Don't expect perfection

It is very unlikely that this tiny project, anxiously typed up over a weekend is going to
be anywhere close to production ready, and marking candidates down because it
isn't perfect is insanely unreasonable.

The only people who could potentially build a prod ready toy tech test project
are those who have solved the exact same problem before, while at work, and with
a team. These people can then basically start coding right away and sleepwalk
through most of it. This is a pretty rare situation, and it will pretty much
be just the very senior seniors who can do this.

The majority of tech tests present brand new problems to candidates, who
will spend the first couple of hours not coding at all, but frantically googling
for tutorials and stuff.

## 4. Don't go out of your way to find mistakes

Related to the above. If the candidate is returning a `200` instead of a `201`,
or if they are using an `int` rather than an `int64` just... chill out.
Those are not insurmountable mistakes.

A good rule of thumb is: if someone on your team today made a similar mistake, and you could
correct it by writing a PR comment in under 30 seconds, then it is something you
can safely ignore in a candidate review.

For larger "mistakes" (eg they didn't apply a design pattern when it would have made
sense to, or their code is under-refactored), consider how much time it would take
to educate them away from doing it again. If you are hiring junior-mid levels, then
it is because your team has capacity to teach people things like this. If you genuinely
don't feel you have time to guide a team-member, then why are you hiring for that
role in the first place?

## 5. Do appreciate the stuff they did well

Because honestly those are the only bits that matter. If everything else is kindof
meh but you find yourself thinking "wow that is the easiest-to-follow bit of recursion
I have ever read" then that is triple points right there.

## 6. Don't be arrogant

_"Oh my god they're using named return variables and they are not inlining their
single `err` checks?! Wow that is basic."_

_"Lol how can this idiot not just know that this will result in an integer overflow?"_

Yeh bruh you are really not all that.

## 7. Be realistic with grading

Hiring managers usually say that candidates should spend 3-8 hours on the average
tech test.

Very few will actually stick to this, because a) they know that the reviewer is
probably looking for an unreasonable level of perfection, and b) this is what
actually goes into doing a tech test:

1. Read the spec (2 mins)
1. Read it again, this time taking notes (5-10 mins)
1. Google anywhere between 20-60% of the terminology (30-60 mins)
1. Realise that while you have done part of the problem while in a team, you have never
done the whole thing solo. Scan some tutorials (30-60 mins)
1. Realise that what you did was something slightly different (like the difference between
scrambling an egg and making eggs benedict), find another tutorial (15-30 mins)
1. Okay you think you know what is being asked now, read the spec again, just to be sure (5 mins)
1. Scribble down a plan. Realise you may not have time to get it all so prioritise some features (30 mins)
1. Start bootstrapping the project. New github repo; install the latest version of language X;
make sure you can run something basic with that language; get whatever dependency tool set up;
commit all that; create your first test file; check that `assert true == true` can run;
get a simple file structure up; nothing more complex than `hello world`; compile/run;
deal with some inevitable run/compile/dep issues; commit. Okay now we are good. (10-60 mins)
1. Start writing the code. For now just get it working, copy-pasta from those tutorials
and stack overflow (1-3 hours)
1. Okay now you actually understand how to do the thing. Go back to your plan, re-write bits (20 mins)
1. Write the code again, but this time make it fashion. Objects, modules, packages, interfaces, the works.
All sharp. (2-3 hours)
1. Write all the tests, integration and unit levels (2-3 hours, whether you actually test before, during or after, the time still counts)
1. Get held up by bugs in the code, by mistakes in the tests, why the fuck is that json marshalling wrong??! (30 mins - 2 hours)
1. It works, it looks tidy, the tests run, read through everything to double check,
change your mind on that interface name. Oh shit the README! (10-30 mins)
1. Write up the readme. Explain what it is, how it works, how to use it (oh fuck do I need a Makefile?),
the choices you made about design. Answer any questions included in the spec because
for some reason they worried you didn't have enough to do. Don't forget the table
of contents. (1-2 hours)
1. Add a makefile and some developer love (30 mins)
1. Go back and rebase and/or squash all the commits. Make it look like you had a rock solid
plan all along and that you stuck to it. Write a shit-ton of context in each commit message. (30 mins)
1. Read the spec one last time (5 mins)
1. Sleep on it
1. Read everything again in the morning, find a couple of tiny mistakes (30 mins)
1. Click send

So yeh. At a minimum the average conscientious candidate will be spending ~>10 hours on
something which may well be new to them, while presenting it in the perfect OSS repo.
I would bet that most candidates don't submit when their test is "perfect" and ready,
but because they are sick and tired of looking at it.

But you know this, you've been there. So be generous and grateful that it does actually
get beyond the `hello world` stage.

## 8. Look for the things you would actually care about if they were on your team

Remember that pretty much any of the technical stuff can be taught on the job,
usually with a 2 min chat and a link to a doc. What you are looking for are the
things which can't be taught, which make someone a pleasure to work with and the
team stronger. Granted, these are harder qualities to gauge when all you have is
a github repo, but repo style is a pretty good indicator.

Do they communicate ideas well in the readme? Can you navigate their code easily?
Have they left comments to guide you through gnarly sections of code (better: have
they acknowledged that the gnarly bits could do with improvements to make them less so)?
Have they written thorough and clear commit messages?

Basically, have they obviously thought about the readers of their code? Those who,
if they were on a team, would end up maintaining it? Have they tried to make that
job as easy as possible for you?

I would put up with quite a few dodgy interface boundaries if they left me the tools
to fix and maintain them easily.

## 9. Don't mark down because they don't do something the way you/your team does them

How the fuck would they know how to do things the way your team does? This is a
actual reason I have seen companies reject candidates for and it is the dumbest
thing I have ever heard of. smh.

## 10. Take into account where they are in their career and what they say they can do

I don't usually start out by reading their CV, but towards the middle/end of my review
I will have made some educated guesses about the candidate that I will want to verify.

Some example situations:

> (Junior-Mid level role): The candidate has made some attempts at writing tests,
but the are whole packages with zero tests and some of the ones which are there
either don't run or don't actually test outcomes.

From this I would guess that they were new to testing, and had only tried here
because the task asked for it. If I check their CV/github and find I am right,
then I am lenient: they gave it a shot and this is the sort of thing they
can learn on the job.
If I am wrong, and they have "TDD expert" or something on their CV, then this is
not a good look for them.

> (Senior level role): The candidate has presented some decent clean code.
While they have missed a couple of best practices and made some choices which
look weird to me, all the other best practices for language X are there.

From this I would guess that they were a new-ish senior. If I am
right, then that is a good hire for a new senior which we can grow in the company.
If I see on their CV that they have never actually worked in language X before,
then this is a more senior person than I guessed, as they have proved that they
can pick up new stuff and apply it pretty smoothly.

> (Junior/entry level role): The candidate has got the thing working, but it is
all in one function in `main`, basically just a big bowl of spaghetti. Naming is
pretty unclear, but they've clearly been hitting youtube and stackoverflow hard
and can demonstrate some interesting tricks and understanding of concepts.

From this I would guess that the candidate is brand new, looking for their first
job. Which is fine, that is the job description. At this level I am looking for
curiosity and ability to learn. If that code was presented for a senior level
role then yeh, I would be writing up a polite rejection letter (with feedback of course),
but in this case I can forgive a lot.

> (Mid-Senior level role): It all works, is clean and easy to read... but some bits
look really familiar. After a quick search, I find some parts came from this very
detailed stackoverflow answer, that method came from this github project, and this
entire package is lifted from a repo in gitlab.

From this I would guess that this person is Senior enough that they know how to
find the information they need, they understand how what they find is applicable
and changeable for the task at hand, and they have no interest in wasting a full
weekend coding a thing from scratch when they are perfectly qualified and know
I, the reviewer, will spend about as much time reviewing as they did copy-pasting.

I would totally hire this person.

If they simply clone another candidate's repo, then obviously that is another
matter, but here they clearly knew what they wanted and took a bunch of different
answers to make it happen.

## 11. The main hiring criteria is potential

People don't get a new job because they already know how to do everything the job
wants them to do. They get a new job because it will specifically let them learn
something they have little to no experience in. Honestly I would be worried about
a candidate who could perform and answer everything perfectly, because they would
probably get bored and leave within a year.

So really you are not looking for things they can do now, you are looking for things
they could do later. Candidates who have a pretty decent stab at a task they
have never done before, or in a language they have never used, are going to handle
your standard (probably staggeringly boring) CRUD api just fine.

## 12. Coding a project from scratch is not what the job is anyway

The actual job:
- Finding bugs
- Reading legacy code
- Reviewing PRs
- Looking up answers on stackoverflow
- Waiting for builds to run
- Installing the latest unnecessary thing that IT/security require
- Waiting even longer for builds to run now that all the CPU is occupied
- Talking about code/design
- Collaborating with people
- Meetings

Basically, you can afford to be lenient on their take-home test.

&nbsp;

---------------------------------------------

&nbsp;

If you got this far, thanks for reading! ttyl

[tt]: /blog/how-to-tech-test
