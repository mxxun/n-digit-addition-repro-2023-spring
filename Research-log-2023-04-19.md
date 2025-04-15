# 2023-04-19

1. ok, let's sketch some....action points?
    1. ~~mildly summarize yesterday's~~
    1. S: go over yesterday's notes & lift some transferrable techniques!
        1. (write down the equation; maybe look @ the trans-circuits for how the equation is supposed to work; simplify the equation to load-bearing matrices by multiplying out linear parts)
        1. cache weights; visualize them
            1. scales fine to [input, output, feature]-shaped tensors, features can be the animation dimension
        1. cache activations as functions of some low-dim subset of input; visualize that
            1. doesn't....scale very well to high-dimensional input, sadly
        1. smth smth logit-lens @ the residual stream on....various inputs, in various layers?
        1. basic concept of interpreting-features (aka what outputs of nonlinearities can be said to mean) & interpreting-circuits (how nonlinearity + weights translates previous-layer feature into next-layer feature)
        1. ......architecture ablation! can the net learn the same thing w/ less layers? w/o MLP layer? etc.
    1. lift yesterday's & today's suggestions
        1. ....I'm even willing to prioritize them some, by their potential
        1. ok, suggestions:
            1. continue w/ yesterday's reading of the paper
            1. ....grind the hands-on skill of extracting & plotting data + visualizations
                1. ...activations & weights sound metaphorically hard, but our idea of plotting logprobs of digits-as-offset-from-naive-sum by now feels simple & familiar; we could Just Do it I think
            1. _skim_ the rest of the paper in hope of extractable higher-level techniques
            1. skim AJ papers in hope of extractable higher-level techniques
                1. especially interpret-in-the-wild ones; they sound like ought to have some....ways to discover interesting circuits or something
            1. start on the analysis-techniques papers, like trans-circuits
            1. find a formal-definition/paper of the excluded-loss technique, get....a better idea how to apply it to discover interpretability-interesting facts (interesting neurons?) about the NN, ....continue somewhere from there (apply maybe)
            1. ~~look into the gradient spike we discovered yesterday~~ not important enough; we can make it go away & that's enough for now.
            1. ...meta: more sketching of the quest tree
                1. (at least look into already-existing online tools about this)
        1. .....pruning:
            1. hands-on extraction & plotting & plotting-logprobs-of-digits-as-offset-from-naive-sum are Shiny; let's do that
            1. excluded-loss seems like it might synergize w/ further reading; it's mentioned _just_ ahead of where we stopped
            1. skimming AJ papers....might populate our pool of "high-level techniques that are likely to be useful, & so we could read up on them & try them out"
                1. so far we have....basically just Excluded Loss, a bunch of low-level we dunno how to synthesize into a high-level approach yet, and Trans Circuits paper, which is....promising in terms of being a framework, but only promising in terms of specific tech by virtue of _probably_ having specific applications
                1. ----btw, NN's gdocs & other pages almost probably have indexes of other named techniques; we could lift these from e.g. MI guide gdoc
                1. ...no it doesn't, but https://www.alignmentforum.org/s/yivyHaCAmMJ3CqSyj/p/btasQF7wiCYPsr5qw (200 COP in MI: Techniques, Tooling and Automation) does mention a bunch of named techniques: Logit Lens, Causal Scrubbing, 
    1. ....& from there I think we'll pick something & get on with it.