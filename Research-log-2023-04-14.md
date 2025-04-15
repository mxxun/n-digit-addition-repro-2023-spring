# 2023-04-14

## quest nodes for AJ
2. Proposal 2 (lift from our extant notes)
    1. ....extract understandings we had in the first play notebooks, so we do not need to learn them again?
    2. continue from [[mechanistic-interpretability-first-steps]] where we stopped before trying Transformer-based NNs at n-digit addition & modular addition
    3. ...& then we could continue trying to reproduce the original paper / 2 papers about 2-digit addition & modular addition

3. hmmmm. ok, honestly I think continuing from the last time sounds sensible
    1. figure how to have Transformers training
    2. reproduce 2-digit addition or modular addition on transformers (it've _got_ to work, right? right?)
        1. `1L Transformer trained to do addition mod 113, trained on 30% of the 1132 pairs`: so that should be enough!! 15k epochs to grokking tho.
        2. `1 2 3 4 5 + 6 7 8 9 0 = 8 0 2 3 5 (where each digit and sign is a separate token). `
        3. `batch size 256, AdamW, weight_decay=0.5`
        4. 1L or 2L? from comments in the colab noteboiok, width=256 works better than 128
    3. go on @ the original paper, checking where we do & don't understand what's going on, & at some point chew through how-transformers-work
    4. (& eventually continue to chewing on more-bleeding-edge papers, getting their techniques, & etc.)
4. ok, that sounds like a plan! kinda.

## ok, how to repro 2-digit addition properly
1. ......possibly 1st big question: what's the _task_ the NN was trained on
    1. (i.e. I have the arch & hyperparams above, & input format also kinda, but I don't understand output & scoring)
    2. https://github.com/neelnanda-io/Grokking/blob/main/figs/5_digit_add_infinite_per_token.pdf suggests that the loss _is_ the sum / avg of pure predict-losses on the last 5 tokens, which. huh.
    3. (but are these predicted sequentially or separately? ......I'd guess separately....)
    4. ....it's very plausible that we should read up on transformer structure & things will get much more sensible
    5. .....I'm now guessing that they're predicted ≈independently; in that _actually_ before the last token-fixing step a transformer model typically has a.....p-distribution over the following sequence, and we can extract its expectations of specific tokens at all(?) positions
    6. (is it a separable distribution over individual tokens, or a joint distribution over sequences? .....feels like "joint" is overdetermined, but we still can measure individual-token sections?)

2. ----separate hypothesis about 2-digit addition: what-are I think simplest parts of it to generalize
    1. (continued in [[mechanistic-interpretability-n-digit-addition]])
