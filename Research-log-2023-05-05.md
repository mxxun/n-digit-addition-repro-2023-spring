# 2023-05-05

1. ok, let's do some more MI things
    1. what to do!
        1. there's the papers gsheet; we can finish the starter-resources I think
        1. then we can go watch the first couple of videos about TS circuits
        1. and play w/ some tools like redwood's! & I do mean play, TG did that out of raw playfulness & we didn't! 
        1. & maybe collect & visualize confusion matrices & see wtf's going on with the above-log2-shelf in some designs
1. ok, that's a plan, let's do that
    1. papers gsheet: digestible research & starter projects left unmarked
    1. except digestibles we put in; now, about starter projects
starter projects:
- activation atlas: already in the table; let's note that jupyter notebooks are playable-with
- BertViz: looks interesting and potentially fun! visualizes attention! interfaces w/ notebooks! a must have
- EasyTransformer: yea let's look @ it, especially @ the demo notebook
- Converting Neural Networks to graphs: meh? meh.

Reviewing explainability tools: uuuh another section splitting from "other"; and actually let's pause chewing on starter-resources here; there's already plenty of neat things

- ELI5: ok, a lib w/ many approaches to debugging classifiers might be A Thing

1. ok, so, walkthrough:
    1. see [[a-mathematical-framework-for-transformer-circuits]]

2. more playing w/ n-digit:
    1. Q: what happens if we disable MLP?
    2. A: it ceases to reproduce, basically? or rather, 1-layer 128/8 model for the 3rd significant digit goes to log2 by around 1k epoch, then lower than that @ around 2k epochs, but very slowly.
    3. Q2: .....what if we add 2nd attention layer.
        1. A: takes even longer to drop below log10 (until 1260), but then finds its way below log2 fairly quickly (1400), does its usual random dance w/ drift down, finds the linear downward path by 4k, & even runs into a spike eventually.
        2. ....so that's fun, but Iunno if I want to do these; it seems that 1-layer-TS-with-MLP is analyzable in principle? ...though the fact that apparently MLP layer is doing considerable work is...maybe not news...eh; we'll see.
    4. ---incidentally: I notice that naive-sum+2 & +9 are both around 2log10 for the length of log2-shelf; coincidence?
        1. probably; on 4th digit it's around 4.5, not 4.6
    5. ---observation: on the 2nd digit, naive-sum+1 and +0 are not oscillating around shared middlepoint of log2! +1 is stably above +0, around the same as loss, which is notably above log2, yes.
        1. & this persists all the way to fully trained state, implying that the difference is data-based.
        2. ...ok, Q: what does the addition table for the 1nd digit look like, to compute how often overflow happens.
        3. it _should_ be: 45×0–8, 45×10–18, 10×9 which.....is not an overflow!! there are 55 nonoverflows and 45 overflows!
        4. so....if we know it's 11 for x to 9 for x+1, what do we predict & what's our loss?
        5. I assume we predict, well, 11:9, but our loss would then be...veeery slightly less than log2, so wtf?
        6. the loss _looks_ like -log 9/20