# 2023-04-17

1. more fucking with the n-digit-addition repro!
    1. (there were a lot of things, but ultimately it trains!!!)

1. ---well, now it trains, except the training graph on 2k batches is......bulllllllshit.
    1. it jumps all the way up to ≈5 every...I dunno how often. periodically.
    1. & it should've converged over 2000 batch steps; NN's did.
    1. well.......what _can_ we do to maybe alleviate this.
        1. weight_decay < 1? 1 sounds....a lot.
    1. ...WD 0.5, le=1e-4 give no effect.
    1. ...what if we disable LR scheduler, what then.
        1. same shit.
    1. le=1e-5.....does not remove spikes I don't think but...at least makes them less prominent because convergence is slower?

    1. ok, original says: `A significant issue in initial investigation of this is that the model had frequent and significant loss spikes (in both the training and test loss). This seems to be downstream of at least two factors: randomisation of data due to SGD (resulting in occasional unlucky batches) and a float32 underflow on F.log_softmax`
        1. `I used full batch training to deal with the first factor, and the second factor can be resolved by casting the logits to float64 before applying F.log_softmax`
        1. so let's try......huge batches, or something.
        1. batch_size 512 ~> 4096: same bullshit frankly
            1. .....I suppose it does not spike above 1.6 now? well......better except not really.
        1. batch_size 2 ** 12 ~> 2 ** 14: 
            1. .....less spikes? and slower though; 
            1. let's put lr back to 1e-3. .....aaaand spikes are back in force. aaaargh!
        1. ...google says that's....a known behaviour of optimizers??
        1. batch_size 2 ** 16: ....ok, that's......something; spikes got down to like 0.64-0.87, that's better.
        1. .....I have a....feeling....that we could slow LR way down and get rid of spikes this way? buuuut maybe not.
        
    1. ok, let's estimate how much data we have actually.
        1. by default, 10 ** 8 examples out of potential 10 ** 10.
        1. 2 ** 20 is ≈ 10 ** 6; 100 batches worth of that will in fact exhaust our prepared data.
        1. _and_ NN says hyperparams for that actually were the same & `batch size 256, AdamW, weight_decay=0.5`! what the fuck!
    1. --ok, what if we set d_model to 256, as someone intended?
        1. ---and accidentally LR to 1e-5: well spikes are still there! it's just kind of...more obvious that the learning between them is slow.
        1. .....actually, let's demonstratively set LR to 1e-10 & see what happens.
        1. _yeah_ that's a clear pattern! _something_ mysteriously bullshit is happening every, like, 39-ish batches.
    1. ...Q: what happens if we fix the data in place? like, keep it training on the first batch forever? lr=1e-10 should prevent any actual _learning_ and we might see if it's data-fuckery or training-fuckery...
        1. I think jumps effectively stop happening? there're tiny steps, but I think they're batch-size independent; might well be something separate.
    1. writing down indices where fucking jumps happen
        1. batch_size = 256: 116-118, 155-157, 194-196, 233-235, 272-274, (311-313, 350-352, 389-391, 428-430, 467-469? yep almost exactly as predicted.) step=39.
        1. batch_size=128: twice as slow, ish: 233-235, 311-313; step=78? 389-391? +; 467-469? yeah.
        1. ok, that.....implies _something_ in the data, for sure.
        1. batch_size=512: yea jumps are twice as fast.
        1. 32: 937, 1249–1250, 1562, 1874-1875. hm, getting closer together.
        1. 8: 3749-3750, 4999-5000.
            1. that's....what, 1250 between? ..............._10k entries between?_ **10 K ENTRIES BETWEEN, YOU SAY?!**
    1. ok, Q: how low in batch_size we can get and still get visible results?
        1. A: .....I mean epochs speed up proportionally I think? 2**5 @ 2k batches is easy...
        1. ....wait, I think the spikes do not depend on d_model. yeah, let's scale that down to nothing & look @ tiny batches; like, size=1 or something.
            1. ---sadly doesn't work, it's same speed at 128 as on 4.
    1. ........ok, now I'm thinking that maybe I should've generated 10 ** 8 _pairs_ instead of 10**4 elements of a pair.
        1. ----aaaaand yeah the above confirms it exactly. ....bet it overfits on the constant 2nd term. or something.
    1. --ok, 3 solutions to this: find some way for the dataset to shuffle the 100M examples, or do it ourselves, _or_ properly generate 200M random numbers and use those.
        1. ....honestly the latter is the best & will simplify the original dataset-wrangling a little.
    1. aaaaand spikes are gone!! ok, let's get LR back to e-3 & see.
        1. YAAAAASSSSS the prophesized graph of the 1st digit!!!!
        1. ---actually, now let's do that w/ lr=1e-4 so it oscillates less in the tail.
        1. .....I kinda think that the scheduler speeds the optimizer up, proportionally to step/10, instead of slowing it down? ...let's put a halve-each-500-steps term in there & see how that works.
        1. ....frankly I'd like to scale it _by loss directly_; LR e-3 is fine up to e.g. loss=0.1, & then we want to slow down like tenfold
        1. ........._or_ maybe we don't, because the vibration of loss from there is....not LR-mediated, but rather data-mediated. because even setting LR to e-30 does not remove the vibration, only makes it flat.
    1. ....I wonder if WD is operative, & if removing it would...let it remember rare occasions where it figures out further improvements on the, uh, logloss=-3.5?
        1. ....it _might_ have let it achieve extreme records of loss on individual batches? .5 micro, e.g., over 3k batches. on WD 0.5 the lowest loss that happens is 17 micro, on WD 1 it's 1.6e-3.
        1. ....& I think the vibrating average is also higher on higher WD? IIRC it literally makes all the weights decay, and GD needs to fight against that somewhat directly, and.....may not have....enough force/precision to simultaneously descend further.
        1. on WD 1 the 3k-batches loss is 0.02ish, on WD 0.5 it successfully dips to 0.005ish, on WD 0 it's...0.003-0.005ish? not particularly lower.
    1. Q: what if we squeeze the model to e.g. d_model=4?
        1. A1: it takes it longer to get to the log2....phase transition...but get there it does, and it certainly keeps going down from there. tho in 4k batches it only gets down to 0.01ish; maybe 0.005
        1. Q2: ok, and if we scale down to 1 head instead of 4?
        1. A2: _now_ it spends _a while_ on the log2 plateau!! & that's cool & convenient honestly!! & still descends to 0.01ish.
        1. Q3: and if d_model is 1 now?
        1. A3: it......_eventually_ breaks through the log2 barrier, but veeeeery slowly; 4k are not enough to even demonstrate things after.
        1. Q4: ..ok, another extreme: width=128, 128 heads?
        1. A4: well! _that_ manages to descend below 0.001 _and_ achieve entirely ridiculous scores of 37e-9 (at the very lowest)! ---that's probably luck, I think?
        1. A4.2: --tho it also sits for whole 100 batches on the log2 boundary, that's interesting.
    1. Q: what happens if we set y to be 5th digit instead of the 1st?
        1. A: I expect fast-ish convergence to 0-ish loss; let's see
        1. A2: ....well I should've been more specific: _actually_ it sits for some time at the log10, then drops to like 150e-6, & then goes as close to 0 as gradient descent will let it; yeah.
        1. Q2: ....and what about next-to-last digit? it _should be possible_ to learn it in entirety I think...
            1. A: ...ok, our predictions are: a log10 barrier, a log2 barrier, and then...to 0? maybe? but w/ significantly more noise than the last digit.
            1. A2: reality: no meaningful log2 barrier! _some_ weird bullshit during 600th-850th batches, but after that, yeah, full descent.
                1. ----ok, actually, not quite true: there's _some_ meaningful barrier after the log10 drop; it's just....a little higher than log2; at around 0.8-0.7.
        1. Q3: ...and what about third-to-last? would _it_ have log2 barrier?
            1. A: prediction: ...log10 barrier, somewhat-above-log2 barrier...then _maybe_ 0.1log2 barrier but I'm not at all sure honestly? and eventually _maybe_ descent to 0; 4k×256 examples is plenty to observe all the last-3-digits.
            1. A2: no descent to 0 at all!! log10 yes, log2-ish yes, 0.1log2 Iunno, maybe, it's too noisy there.
            1. A3: ....though I wonder if, perhaps, a strict conditional on 4 digits is Actually Too Hard for it to implement, whereas same on 2 digits isn't.