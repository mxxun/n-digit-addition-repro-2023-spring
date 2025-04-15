# 2023-04-16

1. ok, so, while there's no-one here, let's recap what we had yesterday & what we need to progress

1. Plan + details
    1. sketch current....plan of progress, perhaps: +
        1. previous steps:
            1. set up an MLP for MNIST, verifying basic pipeline abilities
            1. set up a wrong repro of modular addition; played some w/ params & optimizers; got some experience & some understanding of what hyperparams are better; got hands-on, like, playing-with-training-params experience & setting-up-novel-training-runs experience
            1. listened to the 1st transformer lecture; got a much better (but still not perfectly technical) understanding of Transformer & of embedding-unembedding
            1. read up some more on specific setup of modular-addition; Understood Properly how the input-output & loss were set up (p-sized token dictionary)
        1. current step:
            1. trying to set up a properly faithful repro of n-digit addition; this is Hard to do from scratch, so we're doing piecewise approximations
            1. current target: NN's definitions of transformer & etc copied over w/ only cosmetic changes (his are specialized to modulars); we're targeting prediction of only-a-single-digit to verify all this shit actually works
            1. next target: same, but multi-digit
                1. ==Q:==: I bet it does matter whether we train on sequential-prediction task or on "predict all 6 tokens simultaneously in parallel" task. ....the second is easier to do tbh.
            1. (next target, maybe: install our own understanding of how to do transformers)
            1. (next target, invisible from here yet: crash headfirst into an inability to verify hypotheses properly, & go looking for...transformer circuits paper, or something like that)
    1. sketch current tip?
        1. ok, current tip is "do a train run on the n-digit-addition w/ a single-digit prediction", which requires:
        1. finalizing dataset generation
        1. making friends between our previous dataset setup + feeding-it-into-the-net & NN's
        1. adapting full_loss which plugs into the dataset setup
        1. ....well, running w/ a debug loop.
    1. ....babble what we need to hack together, & then go hack probably

1. .....ok, that...sounds like 80% of a plan. we might need some more elucidation, but for now, fairly solid.

1. code code code
    1. needed changes, take 1:
        1. ~~change full_loss interface to accept X, y~~
        1. ~~change full_loss logic likewise~~
            1. (except not quite clear yet on the expected format/shape of X & y, see below)
        1. ~~change train logic to pass X & y~~
            1. except need loader, see below
        1. change gen logic to produce X & y
    1. needed changes, take 2
        1. copy over our code for loader & adapt it to produce X & ys we need from manually generated pairs
        1. Q: what's the expected format of X & y
            1. for y: see its usage in cross_entropy_high_precision; it's a [batch]-tensor of indices of correct labels; CEHP does, as you'd expect, cross-entropy of log-probs against that.
                1. (that's what torch.gather does; loss then averages over batch.)
            1. for X: uh
                1. I think its principal user is Transformer.forward, which does self.embed w/ it first; see what embed does
                1. embed is Embed(d_vocab, d_model)
                1. and _that_ is doing: `torch.einsum('dbp -> bpd', self.W_E[:, x])`, and W_E is `nn.Parameter(torch.randn(d_model, d_vocab)/np.sqrt(d_model))`, which, uh, what the fuck is going on here?
                1. .....ok, from W_E I start to suspect that X is supposed to be an, uh, _index_ of the token? & then the W_E[:, x] does effectively the same as encoding the token 1-hotly & then W_E-rotating it into....something.
                1. though it's also possible that X is supposed to be a sequence of token-indexes, & that W_E is then broadcast into something?
            1. for X 2: let's see how exactly they're defined & passed in the original
                1. train, test = gen_train_test, & that's pairs[:div], and that's [(i, j, 113) for i in range() for j in range()]
                1. ....so what the fuck, exactly?
                1. ....it _looks_ like it's just batch×3, but......where's the...plus....
                1. _wait_ is 113 the symbol for plus. metaphorically.
                    1. no! for `=`. it's just an arbitrary p+1st symbol
        1. Q2: how do we adapt format-we-have into the requisite one