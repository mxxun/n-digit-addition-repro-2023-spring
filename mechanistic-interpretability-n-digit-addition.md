# mechanistic interpretability: n-digit addition

## meta
1. note purpose/structure
    1. as of yet a dump of all things relevant to n-digit addition repro attempts
    2. probably should become more of an index of subnotes as it grows

### navigation
1. up: [[mechanistic-interpretability]]
2. side: [[mechanistic-interpretability-first-steps]]

### external links index
1. put on github (unsynced copy): https://github.com/mxxun/n-digit-addition-repro-2023-spring
2. the colab in which we worked: https://colab.research.google.com/drive/1OL7lunaj-U2zylnrxuXCChpJBGSjSp9J

## WTDH
1. meta: further work on properly structuring this note in the image of a good subproject note
    1. cut out subprojects that can go into their own subnotes
    1. index hypotheses & curiosities
        1. (probably need to refactor extant hypotheses section; it has a number of obsolete mistakes)
    1. index findings probably

## Subprojects
### above-expected loss shelf of the 2nd digit
1. noticed at the end of [[Research-log-2023-04-17]]
2. [[Research-log-2023-05-05]]-ish: realized that the 2nd digit has 11:9 probs of +0:+1, & that the loss aligns w/ log 20/9 (which P(+1) looks like); got left confused about it (shouldn't it be 9/20 log 20/9 + 11/20 log 20/11, a little below log2?)
    1. ([[Research-log-2023-05-07]] tried confusion matrix; did not get less confused from it)
3. 2023-05-21 (doc'd on [[Research-log-2023-05-22]]): experiment: what if we run the model on 2 datasets, pure +0- and pure +1-dataset, and compute avg loss, P(+0) and P(+1) on them?
    1. prediction: avg-loss log20/11 on +0 & log20/9 on +1, P(+0) & P(+1) 0.55 and 0.45 on both
    2. deranged prediction: avg-loss log20/9 on both for some reason, P(+0,1) = ???
    3. actuality: losses weren't log20/11 ≈0.598 & log20/9 ≈0.799, but instead 0.630 & 0.890; P(+0,1) are ≈0.55 and ≈0.45 as predicted (a little lower for both, possibly more-lower for 0.45 but idk)
    4. this implies that there's some additional.....source of loss.....which is present on both dsets, which we dunno how to diagnose yet. (tho it's..maybe bigger on +1-only dset? but maybe not; our dsets are smol enough there's considerable jitter)
## various smaller projects
1. other potential observations via naive-sum plots
    1. plot not only naive digit, but also "sum if we consider 1-depth overflow, but not 2-depth" and "sum if we consider 2-depth overflow"; these might also have shelves of probability & might signify distinguishable ≈phases of learning (where it e.g. learned to do 2 overflows.)
    1. ...also, in other bases, graph of P(naive-sum+0/1) might or might not signify phases where overflow is learned/not learned/partial??
1. Q: when does the log10->log2 descent happen, as function of base & y_digit & maybe digits?
    1. (at 2023-07-15 hackathon the @vladithur tried to detect shelves, had some progress, did not got to making it work?)

1. ^^SUGGESTION:^^ do a digitwise mod10 addition!
    1. then loss should descend to 0 properly, & we'll be able to isolate the "memorize the addition table" thing
    1. (*OH* and then we'll be able to e.g. ablate neurons & see which ones contribute to memorization, and...)


4. Q: why the fuck loss spikes happen?
    1. [[Research-log-2023-07-15]]: 
        1. observed in 1-layer digit=3 WD=0, & also in 2-layer same
        2. implemented saving & loading, figured how to plot frobenius norm of the gradient; it tracks loss :< & so is entirely uninformative
    2. next suggestion: maybe apply cute attn-plots before and after??

5. Q: can we interpret MLP-outputs as logit-contributions?
    1. (blocked by: reading TransformerLens for whether it can do that)
6. Q: can we ablate like they do in amfftc, to see which neurons/heads contribute what?
    1. (blocked by: reading TransformerLens for whether it can do that)
7. Q: can we measure cross-layer interaction, like they do to find induction heads?
    1. (blocked by: reading TransformerLens for whether it can do that)

8. (multiple: other loss shelves and not-quite-shelves)
    1. (hyp: is 1/10 log 2, where it gets d-1 carry but is oblivious about d-2 carry, a real shelf anywhere?)
9. theoretical: how does attn+mlp, like, mechanically work to predict the digit? (can it form induction heads? or smth.)
    1. blocked by: how does attn+mlp work in general (see at MI-general)
    2. (see also some ideas above about interpreting MLP-outputs as logits & measuring cross-layer interaction of head-to-neuron)
10. confusion matrix vis. miniproject
    1. suggested ?? when
    2. [[Research-log-2023-05-07]]: figured how to extract data for it properly; didn't yet do that per-epoch or visualize or do a time-series
        1. it was a bit disappointingly noisy; maybe still will be informative in non-shelf regimes
    3. (next up: do per-epoch computation (blocked on saving snapshots I think); figure vis.; figure time-series vis.)
11. (low-prio?) wtf's up with spikes in WD regimes
    1. (noticed @ %%todo: extract day & regimes%%)
    2. up next: get reproducibility; get a better look @ gradients or w/e & try to figure what exactly blows up

12. reproduction (historical)
    1. %%TODO: extract reproduction-only pieces there%%


1. ^^S:^^ do a time-evolution of the confusion matrix!! it's a good tool to look @ the output ditribution shape
    1. Q: what's a good way to collect the relevant data? _ideal_ answer is the full distribution over all datapoints, computed at the fixed NN-state, but that's in fact too much.
    1. A: well we have 128-ish batch size, that's not quite but close to enough
    1. (btw, I think a CM is better done not as "compress logits to choice, plot incidences" but as "plot average-prob, separately for each real-answer")

## Insights (?)
1. previous-digit is predictive of whether +0 or +1 happens even individually!
    1. see [[Research-log-2023-05-22]] but the idea is that O(+0:+1|d) = (10-d) : d (when digit=1, but similarly for higher ones I think)
    1. (%%todo: verify this smh?%%)

## History / logs index
1. [[Research-log-2023-04-14]]: 2nd foray into first-steps; focusing on this particular task
    1. looked @ NN's graphs of loss over digits; generated some hypotheses
    1. realized that transformer structure is Actually Important to understanding how to train; went to read about that; understood some important parts (tokenization in particular)
1. [[Research-log-2023-04-15]]: 
    1. uuuhhhh expanded on the hypotheses below
    1. read more of the details of modular-addition in the colab; got scared of the complexity; .....decided to pretend it's Not That Hard because what does it help to be scared of complexity.
    1. ...broke down the goal of "set up homebrewn repro" into "approach a homebrewn repro in small steps which all give off some results-feeling", starting from "copy things over, adapt to single-digit predict, run that, observe loss" & continuing through multi-digit loss _& then_ through interpretability instruments & maybe eventually homebrewing the transformer.
1. [[Research-log-2023-04-16]]: cont'd, & by the end of it we managed to make it train!!
1. [[Research-log-2023-04-17]]: debugged spikes in the train (bad data gen), & finally got the prophesized individual-digit graphs! played some w/ WD, some w/ width, got basic results on how per-digit graphs look like
1. [[Research-log-2023-04-18]]:
    1. figured how to use a local runtime
    1. experimented w/ breaking observed yesterday's graphs (a bunch of additional observations & varied examples to analyze later)
    1. sketch of further....Proper actions like "learn a better & more powerful MI techniques than 'stare at the loss func graph' and 'play with the arch'"
    1. picked an action point "go through mod-p in details, note techniques"; went through some amount of it; there.....probably were some good general-ish approaches & techniques but maybe not enough ≈named ones / .....magically-working ones which can be Just Applied w/o thinking about specializing them much?
1. [[Research-log-2023-04-19]]
    1. summary of maybe-transferrable (tho low-level) interpretation approaches
        1. ....these should live somewhere more general, maybe
    1. another sketch of further things to do
1. [[Research-log-2023-04-20]]: some more; %%todo extract%%
1. (sick)
1. [[Research-log-2023-05-05]]:
    1. Q: what happens if we disable MLP? 
        1. A: it ceases to reproduce, basically? or rather, 1-layer 128/8 model for the 3rd significant digit goes to log2 by around 1k epoch, then lower than that @ around 2k epochs, but very slowly.
        1. (& a bit more there)
    1. naive-sum+2 & +9 are both around 2log10 for the length of log2-shelf; coincidence?
        1. probably; on 4th digit it's around 4.5, not 4.6
    1. noticed that the 2nd digit has 11:9 probs of +0:+1, & that the loss aligns w/ log 20/9 (which P(+1) looks like); left confused about it
1. [[Research-log-2023-05-07]]-ish: did confusion matrices I think
1. [[Research-log-2023-05-15]] / [[Research-log-2023-05-16]]: meditation on how 1-layer transformer predicting `ab=c` can't approximate joint distribution of c|a,b better than...some ""linear"" combination of individual c|a & c|b distributions, which does not get lower loss than probably the straightforward product of these
2. [[Research-log-2023-05-22]]
3. [[Research-log-2023-07-15]]: something about the hackathon

---

1. sources:
    1. https://www.alignmentforum.org/posts/N6WM6hs7RQMKDhYjB/a-mechanistic-interpretability-analysis-of-grokking
    1. Basically not-analyzed there; there's at most `Preliminary investigation shows that 5 digit addition is using a variant of the trig based algorithm`, which.....is not remotely what I'd guess???
    1. (but then I wouldn't guess that it learns a _number-shaped_ representation from raw tokens, so.....)
        1. ---oh, in modular-addition case the token dictionary was p-sized! ok, that's.....still interesting tbh; then it _can_ learn the entire addition table it's shown, but does need to generalize to the rest, and....where does it get token-number correspondences from, again? (.....unclear.)

## Reproduction Project
1. what we need:
    1. set up the training pipeline:
        1. how to define a model: 80% except we dunno how to plug transformers into the basic structure
        1. how to set up a dataset: uuuhhhh we don't quite understand input-output & loss of transformers yet
            1. (probably the next step is doing the "set up a gpt" exercise)
        1. params:
            1. `1L Transformer trained to do addition mod 113, trained on 30% of the 1132 pairs`: so that should be enough!! 15k epochs to grokking tho.
                1. https://github.com/neelnanda-io/Grokking/blob/main/figs/5_digit_add_infinite_per_token.pdf for 5-digit on infinite data shows mere 100s of steps before the first descent; 
            1. `1 2 3 4 5 + 6 7 8 9 0 = 8 0 2 3 5 (where each digit and sign is a separate token). `
            1. `batch size 256, AdamW, weight_decay=0.5`
            1. width=128 works worse than width=256 apparently
        1. how to train a model on a dataset w/ hyperparams: +; copyable from past notebooks
    1. (collect & graph losses, presumably)
1. (further analysis can be its own project; this needs to be cribbed from the paper)

## Hypotheses
### hypothesis about what specifically it learns in the grokking episodes that happen in the per-digit-loss graph
1. ^^Hyp:^^ the _zeroth_ descent of all of them goes from ≈log 12 to log 10; because initially all tokens are equipossible, but actually the distribution has only 10 equipossible.
    1. which is, 2.485 to 2.303; I _think_ this is the main shelf?...but I'd like to confirm that, & confirm the visible distribution.
    2. _except_ for the 1st digit, which smh almost immideatly descends to ≈2.1, what the fuck!
2. ^^Hyp:^^ first descent of the 0th digit is the realization that the 0th digit is either 0 or 1
    1. which should give loss of log 2? ≈ 0.693.
    2. (that should be trivial to observe/prove)
    3. ......by sight it's somehow _a little lower_ than that, at around 0.6, so wtf???
3. ^^Hyp:^^ smth smth 1-digit addition table at ith position will give you the ith digit correctly 90% of the time; the-sum-90%-the-succ-10% should minimize the loss in this case
    1. which is I think 1/10 log 10 + 9/10 log 10/9 ≈ 0.325
    2. ----correction!! _half_ of the time; overflow happens, statistically, on half of preceding digit-pairs.
    3. technical ^^sub-hyp^^: the important part is the ""formation of an attention head"" that looks specifically at -6th and -12th digits; exactly those whose sum is supposed to happen in the current position
    4. sub-hyp: 100% of the time for the last digit
    5. postdiction: _if_ it in fact learns the addition table in toto, the last digit's loss should go from the base shelf of (log 10?) to 0
    6. kinda-prediction: for the 2nd-to-last digit, it shall go from the base shelf to the next shelf of ~(entropy of ~~0.9-0.1~~ log 2!!)~, and _then_ to 0 at some later point.
        1. ----actually, violated! digits 1–4 stop descending quickly at loss=0.7–0.5!
        2. ---in sight of our error above, un-violated: carries happen 50% of the time, so know-sum-dunno-carry is a flat-distr over 2 possible digits, which has entropy log 2 ≈ 0.693.
    7. kinda-prediction: for the 3rd-to-last and further, there should be further shelves, defined by "can figure contribution of past j digits, is optimal about probabilistic contribution of the further digits"
4. ...how does this hypothesis factor into the 0th digit?
    1. a simpler addition table for "overflows or not"; 100 entries, 45 definitely do not overflow, 45 definitely do, 10 are on thin ice
    2. therefore, it can upgrade from "50% of 0, 50% of 1" to "90% of time, 0 or 1, 10% of time, half-and-half", dropping its average loss to...10% of log 2? which'd be 0.0693. which I....can't reliably distinguish from the real value on the graph; I _think_ the graph goes lower for some reason? but I dunno.
5. ^^Hyp:^^ something _different_ happens if we ask it to output digits from least to most significant

6. ok, uh, things to do about this hypothesis:
    1. _REPRODUCE_ so we actually have our own run to play with, not something that might not even happen again
        1. (optionally, load NN's checkpointed runs & figure how to poink them)
    2. ...verify, smh, that the shelves @ specific points give even odds for a. 0/1 for the 0th, b. 0–9 for others, c. sum / sum+1 for others after appearance of Addition Table
        1. verify smh that the shelves _are_ at levels we expect them to be (log 10 & log 2)
    3. ....locate the addition tables smh?
    4. .....figure what in the fuck digit-1 is doing with its sub-log-10 loss
    5. .....figure wtf digits 1–4 are doing at their 2nd shelf/slower-descent
