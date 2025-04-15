# 2023-05-22

1. ok, let's do something
    1. ~~note down the yesterday's~~
        1. soooo we looked @ how avg-loss & P(+0) & P(+1) look like on only-+0 and only-+1 datasets
        1. and that was _fucked_ because our predictions basically borne out — P(+0) is ≈0.55 and P(+1) is ≈0.45 on both!
        1. however _losses_ on both weren't log20/11 ≈0.598 & log20/9 ≈0.799, but instead 0.630 & 0.890. which implies....some kind of fuckery, where it's getting some additional loss from who knows where.
        1. (which concludes our investigation via separate-datasets. for now.)
    1. ~~part 2: write this down in the relevant location!!~~
    1. ~~part 3: write down TG's idea about 0-digits being informative~~
        1. they are, aren't they? O(+0:+1|d) = 10-d : d, & even if this thing did not yet learn the joint addition table of last-digits to predict +1 perfectly, it can do no-worse-than the distr on one digit, or, probably, the product distr of 2, (10-d1)(10-d2) : d1×d2. (Iiiii think the product is the best linear combination buuuuut I have not proven a theorem about it.)
        1. so....we can expect the net to learn this.
    1. sketch the today's
        1. idea: what if we apply the "individual contribution to logits" to the MLP?
            1. isn't it _effectively_ a bunch of neurons whose contributions are individual, &...which effectively output logit-contributions?
            1. (need to check that that's true first, but then, yeah)
        1. idea 2: can we apply their idea of "let's see how much interaction is going on between 1st- & 2nd-layer heads & focus on those that have nontrivial interactions", but to interactions of attn & mlp?

1. ...ok, maybe the first part we should do is the setup.
    1. dod: TDT, daily plan, current stretch plan

1. daily plan:
    1. ~~put yesterday's in the right place~~
    1. further levelling:
        1. inefficient but fun: try to apply ideas we've read about to our pet project
        1. proper: finish amfftc
        1. do...exercises of amfftc
        1. steal their visualizations of neurons (!!!)
            1. (maybe put me somewhere in global MI projects)
        1. steal their ablation (?)
        1. read/practice about TransformerLens, maybe it does all we want trivially & painlessly
        1. read more about induction heads
1. plan 1
    1. put the yesterday in its place
    1. uuuuhhhh prioritiiiiize the levelling ideas & put some of them somewhere permanent
        1. (maybe install a permanent-ish place for shiny miniproject ideas)
    1. finish the daily plan, maybe put timings on it; probably put dods on it
1. OOF. ok, act 1.
    1. ~~put yesterday's in their place:~~
        1. ~~avg-loss experiment~~
            1. ~~DOCUMENT IT IN THE NOTEBOOK so it's at least remotely reproducible~~
            1. ~~where to put it: MI ~> n-digit ~> wtf's with the loss shelf~~
        1. ~~last-digit-is-predictive: put it in the...same place...~~
            1. but actually it should go into insights. ...ok done.
    1. aaaand that's time.
1. plan 2
    1. prioritize???
    1. maybe figure out where to put all these interesting ideas so they're not missed
        1. probably they can be put to amfftc note's subprojects mostly?
        1. and duplicated in the main MI note, & maybe in n-digit note because they're concretely applicable?
1. act 2
    1. ok so.
        1. trying to apply ideas is virtuous but should wait until we've looked @ TransformerLens & checked whether it does what we want
        1. finishing amfftc _might_ be doable on minimals; let's high-prio that
        1. exercises....are important & might have stealable code; let's look into them
        1. steal visualizations & ablation are bigger projects, & both also wait on checking TL
        1. more induction heads let's put into low-prio
    1. ok, so actual prio:
        1. look through the rest of amfftc & wrap it up
        1. exercises are important; look at them at least & maybe do them also
        1. TL to check if we don't need to steal or to apply ablation or MLP-logit-contribs or pairwise-interaction-measure ourselves
        1. applying ablation & individual-contributions-of-MLP are vitruous ideas but should wait (but be noted)
        1. stealing visualizations & ablation likewise
        1. more induction heads sounds fun but maybe later / if we get bored of all that above.
    1. &&&& I think wrapping up & exercises will be enough to fill the day.
    1. ok, now let's put things into notes:
        1. exercises are basically already there in amfftc note, cool
        1. let's put TL into MI main. (..already there.)
        1. ablation, MLP contribs, visuals, interactions are put where they go
        1. induction-heads paper is put where it goes
1. OK, _now_ we have a daily plan:
    1. finish amfftc (ideally quickly)
    1. look @ amfftc exercises & probably do them
    1. look @ TL, especially with an eye for can it be applied to steal visuals, visualize logit-contribs, do ablation, do interactions

1. ok, still act 2. let's try to finish amfftc in 5m.
    1. ....well we didn't, but _now_ we did finish.
1. plan 3, look @ exercises? or go for a walk?
    1. walks are good, let's walk.
1. plan 4.
    1. ....ok, technically there's....still small-font part of amfftc, let's skim that
        1. `Additional Intuition and Observations`, fuck! ok.
        1. ok fine done.
    1. we had a "look @ amfftc exercises" and "look @ TL"; let's do that.
    1. first: locate...amfftc exercises
        1. https://transformer-circuits.pub/2021/exercises/index.html
    1. second: ...well look through them! see what they offer/train!
        1. [[amfftc-exercises]]
        1. augh they're for people who have not been taking notes and have not tried to think shit through!!! fuck's sake!!
        1. fiiiiiine I guess it's cruel to put up exercises which are strictly harder than the material you present....
1. ....ok, what now.
    1. look at TL, right!
1. ok, plan-act 5: looking at TL.
    1. leeet's locate some foundational page of TL.
        1. https://github.com/neelnanda-io/TransformerLens & also there's a readthedocs.
            1. https://neelnanda-io.github.io/TransformerLens/
        1. https://www.lesswrong.com/posts/hnzHrdqn3nrjveayv/how-to-transformer-mechanistic-interpretability-in-50-lines ok that sounds nice to fuck with if not necessarily comprehensive about TL
        1. there's https://transformerlens-intro.streamlit.app/ w/ in particular exercises and solutions for TL&induction-circuits and for interpreting-algo-model
            1. https://github.com/callummcdougall/TransformerLens-intro is I think the source of that.
    1. ok, let's do https://neelnanda-io.github.io/TransformerLens/ & look for.....things we....wanted to steal.
        1. they were...visuals for attention, interaction matrices, ablation, logit-contribs
        1. heavily inspired by Garcon! & intends to make exploratory research easy! ok that's good.
    1. Gallery:
        1. `Decision Transformer Interpretability: A set of scripts for training decision transformers which uses transformer lens to view intermediate activations, perform attribution and ablations. A write up of the initial work can be found here.` ok sounds good
        1. `Induction Heads Phase Change Replication: A partial replication of In-Context Learning and Induction Heads from Connor Kissane`: sounds not _as_ good as the first but we also want to find induction heads so
    1. https://neelnanda-io.github.io/TransformerLens/content/getting_started.html
        1. there's a `main demo` link & also notebook about Indirect Objection Identification & a recording of that, to look @ how it looks like
        1. ....aaaand that's basically that.
    1. ok, I think the next steps are:
        1. setting up a proper note for TL & projectifying it
        1. ....reading & maybe copying over & playing w/ the main demo, & maybe w/ IOI notebook.
        1. ....both of which Sounds Big so I don't want to do that now. ...because it's late, let's go with that.
