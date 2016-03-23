---
layout: post
title: Strange Loop
category: Code
---

I've known about the conference StrangeLoop for a while, but yesterday I decided to google the top video search results. I never expected how useful they would be! I've watched other academic lectures, but these are a cut above the rest. I'd like to comment on the two I've seen.

## Transducers

Here we have a talk about processes and how to compose them effectively. Rich, the speaker, implements map and filter in terms of transducers. There's a question of primitives here: if we are predisposed to use constructive logic, should transducers be more primitive than map and filter? There's some interesting comments about typing toward the end as well.

https://www.youtube.com/watch?v=6mTbuzafcII

The correspondence of the **>=>** of haskell to transducers excited me, so I began to implement transducers in scheme. Sadly, it turns out licensing dissuades me from directly reading the clojure code, so I'll have to find other sources, or reason out the implementation myself. 

## Propositions as Types

This lecture covers the Curry-Howard Correspondence, so it's of great interest to me. It's accessible and gives a good history of computer science. Of particular interest to me, it provides a few charts linking logic to their correspondence in computing.

https://www.youtube.com/watch?v=IOiZatlZtGU

I've been looking into SK combinators, lambda calculus, and the two's relation to constructive logic and natural deduction, so this talk was especially interesting to me. As a bonus, there's a surprise reveal at the end mirroring Gerald Sussman's stunt in his SICP course.

## More on the horizon

Thus far, I've only watched one other lecture. It's entertaining, and uses tetris to explain how to "defragment" your deployment process. It's not a current area of interest for me, but it is very well done.

https://www.youtube.com/watch?v=pozC9rBvAIs

I'll be watching more (and rewatching the first two). I've certainly underestimated the value of these conferences - there's something enlightening here for all academics.
