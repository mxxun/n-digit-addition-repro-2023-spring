# 2023-04-15

1. ok, we need a plan; maybe to recover the plan from somewhere
1. a plan: go watch / read NN's exercise on defining our own GPT-2, and then Just Do It
    1. which really ought to give us enough competence/understanding to be able to roll our own 5-digit-adder

1. ok, where's that exercise; let's find an url to it, lift a list of action-points from there, and start going over that list

1. ....tho: actually, can we.....just take NN's definition of a Transformer, use that to have our own model, & then fuck around until we have something that works?
    1. (code code code)
    1. OK, having copy-pasted most of the thing into the notebook, we have, I think, most of hard things done away with, except:
        1. full_loss currently has an `fn` hole, computes labels weirdly, & in fact is specialized for the single-label situation, not for the 6-digit-output
        1. .....aaaaand we'll need to do _something_ to make the model generate logits for 6 tokens. probably the std thing where we sample predictions according to probs, append the sampled to the data, predict on _that_, and iterate until we're satisfied
    1. ok, so.......
        1. we _could_ run this...._before_ doing the hard work of iterative prediction of 6 tokens; we can cut the output to a single (0th) token, and run a model on that — and in particular test that it fucking works
        1. (& then)
