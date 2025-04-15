# 2023-07-15
1. there's a hackathon weekend! in principle.

1. ok, so I have 2 plans:
    1. sketch, uh, hackathon-running plan, maybe some resources to crib from when It Comes
        1. maybe...shortlist some Cool Ideas (except there're so fucking many)
        1. maybe continue my own research somewhere (I had things I wanted to crib and practice)
    1. if people come:
        1. brainstorm What We Want To Do
        1. microresearch on topic, find some notebooks to copypaste from
        1. uh, attempt to hack it together

1. ok, ideas for what to do
    1. Q: how many epochs until log2, as function of base and digit?
    1. Q: suppose we know that 3rd and higher digits (in base 10) do not descend to 0 on reasonable scales, if on 1-Block. what if 2-Block?
        1. (repro): ok, yeah, 2nd happens in 3500-ish epochs, 3rd does not in 15k.
        1. let's do 2 blocks, 128/8:
            1. uuuuh it hit logloss -15 at 7.5k-ish epoch, but I don't think it's a true "got it", but rather an especially hard random spike
            1. after lowering logloss floor, it....iunno if it _stabilizes_ at -10? it hits -16 eventually?
            1. ......it _looks_ like it's running into the erstwhile spikes / catastrophic forgetting / w/e, and does not eventually descend to 0 by 15k batches
        1. 128/128?
            1. similar dynamic I think. yeah.
        1. ...remind me, does this spiking happen on 1 block also
            1. ....only sometimes!
            1. I...dunno exactly if those two descent-blips on this new seed are different from the random walk it's doing usually, but they _look_ different
    1. ok, I.....kind of think I should figure out wtf happens in those spikes.
        1. these are 0-WD! let's indeed try to diagnose them.
        1. [+] figure how to save models properly
            1. os.path.join is involved
            1. ...ok, 2M per snapshot and a hundred snapshots per run is...a bunch....of place per run, I don't want to do that every time
            1. but, ok, 
        1. ....reproduce the....exact conditions controllably
            1. I think we'll need to load a. model, b. optimizer, c. scheduler
            1. & d. set dataset to the correct offset??
                1. or maybe the same thing will happen / work if we do a run on the initial data? ...we can investigate, how about.
                1. ...if starting from the beginning, it's _basically_ the same dynamic; on the main run it jumps around 8700, & in the repro from 8600, around 8720, but, eh.
        1. ....that it's not perfectly stable but somewhat stable on different dataset, uh, suggests....something.
            1. ...that we can, in principle, change it by altering the data somewhat, but also that....smh the thing that produces the spike is moderately data-agnostic.
            1. ....the latter sounds Bad, but like, it's not like we now understand _less_ what's happening.
            1. let's figure some additional way to look into this thing.
        1. [x] how do we look into what's happening?
            1. we could, iunno, extract from the optimizer some measure of the gradient scale — I bet gradient is a vector, & it may have e.g. a norm & we can plot that
            1. .....we might need to comprehend what exactly our optimizer does. (AdamW, right? yeah.)
        1. [+] ok, let's try to extract the gradient measure. and plot it, yes.
            1. [+] Q:.......where _are_ they, inside the optimizer? They're somewhere inside the optimizer, right????
                1. A: uuuhhh, .param_groups might be the thing? let's try to look at .param_groups of the optimizer & observe gradients or w/e inside.
                1. ....well, `print(optimizer)` shows optimizer-level properties of a param group 0, & does not show any tensors, but what about .param_groups
                1. oh yeah list w/ a dict w/ ..a list of ~tensors. (of 11 "Parameters")
                1. ....aaaand those have a .grad! awesome!! so we can at least have those.
                1. .....or, as another stackoverflow answer suggests, we could actually address our matrices through the model (which wraps layers, which wrap parameters proper), and those are in fact the same as optimized parameter tensors, so they also have `.grad`s. ok.
            1. well........sure enough we plotted it, and it's _entirely uninformative whatsoever_
            1. because for some reason frobenius norms of those are all basically proportional to loss?? & therefore are tracking all the same thing???
        1. alternative hypothesis: could it be.....overtraining on some kind of natural subset of data? and then, after a hundred batches, a new one violates the subset boundary and it snaps out of it????
            1. this sounds like bullshit, and yet I don't understand what it's doing enough to be confident it's not utter bullshit
        1. (checking obvious things:)
            1. can't do .backward() twice; need to compute a new `train_loss = full_loss(model, X_train, y_train)` to do that again
            1. mere loss.backward() does not alter params; optimizer.step() does. ....both facts are visible when we're comparing print()s of some param group.
    1. uuuuuhhhhhh. can we apply some other instrument to understand what the fuck?
        1. Iunno, look @ attentions before and after the spike?
