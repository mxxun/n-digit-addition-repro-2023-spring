# 2023-04-18

1. following the yesterday's!
    1. first, we might want to observe in more detail the underlying data for our hypotheses on log2 and log10
        1. e.g.: measure & plot p(naive-sum-digit), p(naive-sum-digit+1), and other digits through the lens of delta-from-naive-sum.
        1. ---and also measure the prob of the correct digit---wait no that's just loss. I mean, loss is logprob, not prob, but you know.
    1. second, we might do smth about training on all digits simultaneously
    1. third, even simpler, we can test if our convergence observations on individual stay stable under arch chagnes (less heads, more heads, etc)

1. ....ok, let's do some experiments on the 3rd, w/ changing width, heads, etc
    1. (a sidequest into making this thing run on a local runtime): done
    1. Q1: what does it take to break convergence of the last digit to 0?
        1. I assume WD is enough; how much WD is enough?
            1. reference WD=0: slows descent down at around 200e-6, then linear-going-into-nonlinear descent to at least 60e-12 & probably eventually lower
            1. WD=1: ...well it does something interesting? there's an unmonotonicity at 0.001, then log-linear descent to ≈20e-9, then stability there. weird!
            1. WD=0.5: there's a....brief step at 800e-6? then descent to 8e-9 & stability there-ish; although, curiously, it dips to 7–8 & then slowly goes back to 8–9
            1. WD=0.75: basically interpolates between 1 and 0.5, no surprises
            1. WD=0.25: a brief step / mild growback at 200–300 batches is still visible
            1. WD=0.1: the brief step is not visible as such.
            1. WD=0.2: same! WD=0.25 is about the last point where it's that prominent.
        1. ....I have a _suspicion_ that the slow growback after descending to ≈10e-9 in WD-regime is the same kinda thing as the mild growback after descending to 300e-6, just 30× slower.
        1. ok, anyway, w/o WD: how much do we need to shrink the model to make it unable to descend to 0 in 3k batches?
            1. default-run w/ WD=0, to have a reference point for epochs: first descent at 120, slowdown at 200, gradual slowdown all the way to 3k
                1. & WD=1: first at 140ish, a bump at 200–280ish, lowest at 1600ish
            1. ok, WD 0, d_model=4: uhhhhh ok, that's enough apparently??
                1. it's stable at log10 all the way to 2400ish, and only then begins a slooooowww descent, which does not complete by 5k, & seems likely to continue for at least 6k more
            1. d_model=8: _OK_ that's faster. log10 until 1700ish, 0.8~0.06 slowdown during 2000-3k, & then loglinear to 400e-12 at 5k, where it slows down again.
            1. d_model=16: ...I'm predicting...log10 until longer than 1000 (1100-1300?), again no obvious step for log2, & otherwise similar pattern to d_model=8 & d_model=128.
                1. yeah, log10 until 1200.
            1. ok, now let's shrink num_heads. d_model=16 & 1 head?
                1. descent from log10 much sooner!! @ around 600.
            1. d_model=4 & 1 head: descends from log10, but slowly & late, ~2600
            1. d_model=8 & 1 head: descent by 1800ish, no meaningful slowdown at 0.8, keeps dropping to 0.1 @ 5k & presumably lower.
        1. --ok, so......less heads can smh speed up / make possible convergence on the thin NN. what about more heads.
            1. d_model=128 heads=128: nothing new tbh
            1. d=32 h=32: log10 until 640ish, then descent about as usual
            1. d=8 h=8: log10 until 1600ish, then descent; spike at 4250ish!
            1. d=4 h=4: as above.
        1. Q: can we predict & experimentally verify descent for width=2 & 1?
            1. ok, by prediction, d_model=2 will descend by 4k & d_model=1 by 6k; let's...test these.
            1. d_model=2 h=2: nuh-uh! stable until 5k! ok, fine, I'll try 10k.
            1. 10k: nope!
            1. ...let's see if 3/3 fares better: yep! at 5400ish.
            1. ....fine, let's run 2/2 on 30k batches w/ threshold of 1 while we read or something.
            1. ..._extremely_ eventually, at around 26900, it does stumble into the right thing & starts to descend!!!
            1. given the ×5 blowup from 3/3 to 2/2 I don't really want to try for 1/1; it could well exist somewhere in 1M batches or something.
            1. ....tho it's perhaps notable that there're mere 130 parameters in the 2/2 model!
            1. 2/1: ....descends from log10 faster, @ 5400ish, but then gets stuck higher than even log2?? at, uh, loss of _2_???? what the fuck!!
            1. 1/1: does not descend from log10 in 50k; let's let it be for now.
        1. conclusions:
            1. even small amounts of WD are enough to make a low barrier, _but_ there's an interesting bump that's visible at WD>=0.25, that's cool!
            1. shrinking model exponentially slows it down....also exponentially? I think. smth like ×1.5 time to drop per /2 of d_model.
                1. though there's also quantitative?? slowdown and vibration of descent after that.
                1. ---maybe hyperexponentially, when close to 0: 8/8 takes 1800 to descend from log10, 4/4 takes 2600ish, 3/3, 5400, 2/2, 26900, 1/1 >50k if ever, so.
                1. ==smth smth maybe there're easier paths through 4/3-circuits than directly through 2-circuits...==
                1. ==or maybe chances of extant circuits to be accidentally close enough to the necessary basin of attraction scales....smh w/ the number of potential circuits?==
                1. (well, in any case, that's for if & when we can understand circuits....like at all.)
            1. number of heads....._does_ matter, quantitatively, in that for this task more heads are worse (considerably slow down convergences); maybe 4 heads'd be better if we were trying multiple digits? but for now let's keep it at 1 head, why not.
    1. Q2: what does it take to break convergence of the next-to-last digit to 0?
        1. ...vaguely suspect that exact same things as for the last-digit, but let's test out some things.
        1. ....ok, actually _this time_ dropping from 128/4 to 128/1 breaks the convergence to 0 within 2k batches!
            1. also, WD=0.25 diminishes the loss vibration _drastically_, let's maybe keep this for convenience.
        1. basic shape investigations:
            1. ...ok, at 128/2 & 128/4 and various WD I observe a consistent 0.8-step, then descent, then a 0.001-0.01-0.001 mountain, then descent further. all in 2k-batches range, tho more heads makes things happen faster I think??
            1. yeah, including repeated spikes
        1. -----_ALSO_ the peak is repeatable!! observe 128/4 (WD=0.5, WD>=0.225???) & 5k batches, which peaks thrice in the exact same pattern.
            1. on WD=0 it...repeats only once or twice & then descends to 0 normally; on WD=0.1 also, on WD=0.15 no (or too small) spike happens, on WD=0.2.....one and a half spike????? a small one @ 3k & a big one @ 4k?
            1. on WD=0.225 multiple spikes, ok, fine.
            1. FINE it's probably a gradient spike of unclear provenance. would be nice to figure where it comes from & fix it but maybe Laaaater.
        1. ok, getting back to the question of what does it take to unconverge to 0
            1. 128/2 converges; 128/1 doesn't; 64/2 doesn't; 64/4..does!
            1. 32/4 I expect not to (yep); 32/8 yes to (nope!)
            1. ok, then....32/16 and 32/32 probably also no? ----nope, 32/16 looks converging! spikes are harder but still.
            1. 16/16 I'd bet also nope....but it does! fucker!
            1. 8/8 I don't know what to expect really.
                1. ....sits on log10 for too long, & then descends too slowly to tell if it'll stabilize somewhere lower. @ 15k batches, not-quite-stabilizes at e^-1.5. .....but _then_, around 14k, descends again to e^-2.5! bitch.
            1. ....do let's see if 8/4 and lower will descend faster or what. (all @ 5k batches.)
            1. 8/1: yep to log2. ---a little higher actually but w/e.
            1. 8/2: uh to 2.1 first and to 1.4 second, which uh......
            1. 8/4: .....I _predict_ one or two steps, but......
                1. but actually, no particular stable steps!! more gradual, in 8/8 style. keeps descending, at ~0.73 @ 5k.
        1. ....so: too few heads will do it (and "few" increases w/ decreasing width...); what about too many heads?
            1. 128/128 converges; ok, others probably converge too.
        1. what about 256/1? 512/1?
            1. 256/1: does not converge; 256/2 does
            1. 512/1: ...that's considerably slower. doesn't. 512/2 does.
        1. ....conclusions: 
            1. there's a repeatable gradient spike, intensified by WD ~~but occuring w/o it too~~ _maybe_ not occuring w/o it? 128/4 @ WD0 doesn't produce it for 5k batches; 128/8 neither; but 128/2 does.
            1. ..answering the original, _what it takes_ to make it stop converging to 0 is too few heads! 1 is too few (even for 512/1), & for sufficiently thin width 2 & 4 & 8 are also too few. 16/16 converges, 8/8 doesn't.
    1. Q3: can we make the 2nd-to-last digit converge to 0? can we break convergence to log2, or below?
        1. reference run: 128/4 doesn't do it.
        1. extrapolating from not-enough-heads above, I feel like more heads might help:
            1. will 128/128 work? yep! 
            1. 128/32: also yep! 128/8: also yep. so it's 128/4 specifically that wasn't enough.
        1. ......ok, uh, extending the question: what if we do _further_ digits.
            1. thousands, 128/128: doesn't; 256/256: also doesn't. all right then.

1. ok, at that point....we really should collate these results into some conclusions & followsups _and_ go chew on....better ways to look @ a network beside mere loss graphs.
    1. (maybe make it output quirkily-parametrized digit probs, to confirm or deny the theory that it learns to choose between the two "possible" ones.)

1. ok, things to do:
    1. ....write up smth from yesterday & above to the MI~>n-digit note
    1. sketch the next quest
    1. Optional quest: implement our ideas for introspection into the net, where we measure digits by offset from the naive one
    1. Quest 2: practice some nontrivial MI techniques on the n-digit addition
        1. what is there to practice even?
            1. query the mod-p part for techniques
            1. query hackathon papers
        1. how do we....observe circuits; how do we answer _why_ smth is output'd / that a circuit does calculation like "addition table" / how to locate a circuit that's responsible for the thing we think happens?
            1. ...probably a good idea to read the trans-circuits paper & cross-ref w/ the mod-p details
        1. .....that last has a comprehensible answer in the shape of "ablation" or "activation masking" or "smthinged loss"
    
    1. ok, so.....specific plan points:
        1. ~~write up the above & yesterday, _at least somewhat_~~
        1. go through mod-p details again, note specific techniques w/ links
        1. ....probably at least skim the trans-circuit paper?
        1. .....look for answers to the question "how to observe circuits" / "how to figure which neurons are important" / "how to interpret probably-important neurons"
1. ok, act
    1. mod-p details noting techniques & links
        1. https://colab.research.google.com/drive/1F6_1_cWXE5M7WocUcpQWp3v8z4b1jL20#scrollTo=Modular_Addition_Transformer_Detailed_Analysis
        1. segment filtering: 
            1. overview of the learn algo (not really what we care about)
            1. technical details: let's skim but we got it
            1. initial observations & model simplifications: sounds interesting
            1. intuitive overview of discrete fourier: uh I think I don't care
            1. analysis of the features: yes pls
            1. neuron & attention circuits in the appendices: yes pls
            1. analysis of how circuits develop during training: yes pls!
                1. excluded loss usage: yis!
        1. ok, with that in mind let's go over them.
        1. learn algo overview:
            1. `the embedding W_E extracts cos(wx) & sin(wx) for a couple frequencies, ...[which] looks like the embedding matrix being sparse if changed from vocab basis to the fourier basis`: ok, that's interesting, and I _don't_ want to learn this shit but plausibly I'll need to at some point. ....Later, maybe.
            1. `The activations at position 0 (x) and position 1 (y) are routed to position 2 (=) by the attention heads`: ....I don't understand what this says & I suspect I need to.
            1. `The neuron activations are linear combinations of .... The multiplication is done by the elementwise multiplication with the attention pattern and the ReLU activations.` ...also what.
                1. `This can be directly seen by looking at the neuron activation as a function of (x,y) and seeing that it's well approximated by a linear combination of those 9 terms.`: ....ok I guess? ----yeah, that's extremely comprehensible; we can directly collect activations on a big dataset & approximate this technically much-parametric func with the 9-parametric func & observe that approximation is in fact indicatively good.
            1. `W_out and W_U cancel out all directions in neuron activation space other than those corresponding to cos(w(x+y)),sin(w(x+y)) for each key frequency. This can be directly seen by comparing the coefficients of each term before and after.`
                1. .....I guess, yeah?
        1. model analysis setup: interesting; we'll need to set up snapshotting to do that later; for now we don't care as much.
            1. helper funcs: also interesting! 
        1. Model simplifications:
            1. `The initial value of the residual stream at position 2 (ie between the embedding and attention layer) is constant (the embedding of "=" plus the learned positional embedding of position 2) This is a theoretical observation that follows from the problem set up, the rest are empirical.`: I understood that!! ok cool.
            1. `Position embeddings in position 0 and 1 are the same, because addition is commutative, and just act as bias terms`
                1. yeah I can kinda see how that could be true; don't _quite_ know how to check that, but I think I could figure it out
                1. ...actually, |x-y|²/|x||y| =0.766 does not convince me? or cos(x,y) = 0.617? like.......that's not _too_ close though I admit impressively non-0 for 128-dimensional vectors.
                1. otoh difference in neuron activations.....also isn't quite convincing without absolute values of activations!! there should've been some comparison, yes, just like the comment over there says.
                1. ....I think it might be instructive to randomly permute dimensions of y & compute similarity metrics on that; sample them, even.
            1. `Attention from position 2 to itself is negigible`
                1. ok, I _guess_ this should make sense, & I can finally read the graph a bit (well, values in it, via mouse-over), & indeed all 4 heads have the (attention value) at pos-2 near-0.
                    1. ----or, like. look @ the graph proper through Chromium.
                1. (& I can check/figure how `blocks.0.attn.hook_attn` is the intended value later)
            1. `The contribution via the residual stream from the skip connection between the attention and MLP layers is negigible, it just acts as a bias term rather than doing meaningful computation`
                1. that......I dunno what means, and commenter points out it's not _very_ convincing. ugh.
        1. .....those were important, probably? well-----they probably make the equations that follow simpler, & therefore are motivated from that....
        1. ok, let's look @ equations.
        1. Key Equations
            1. https://transformer-circuits.pub/2021/framework/index.html link!
            1. ok, so...."3 nonlinearities: softmax in the attention, relu in activation, & final softmax" I understand, & can in principle trace through; cool
            1. then 2~>2 attention ≈0 and 2~>1 & 2~>0 symmetry come to play to make a matrix look like (Row, 1-Row, 0), & that simplifies the softmax to a sigmoid, which uuuuuhhhhh not really something we'd have in our, or in arbitrary model....
            1. ...but _maybe_ most of this is irrelevant, and "there are 3 effective matrices that fully determine the model" is!
            1. and plotting these, even in a naive basis, yeah, demonstrates how they're Obviously Periodic.
        1. Circuit and Feature Analysis
            1. `interpreting the features represented by non-linear activations (output probabilities, attention patterns, neuron activations), and interpreting the circuits that calculate each feature (the weights representing the computation done to convert earlier features into later features)`: ok, i...can understand that...
            1. `In this section we interpret the embedding, the neuron activations (a feature) and the logit computation (a circuit). These are the most important bits to interpret, I explore Attention Computation and Neuron Computation in Appendices (these are fun but messier, and not essential to understand grokking)`: ok I GUESS, except that doesn't sound like it'd help w/ discovering circuits? ......ok, fine, maybe it would, and maybe understanding the...features....that neurons represent would be helpful, yes.
            1. neuron activations:
                1. `TLDR: Each neuron's activations are a linear combination of the constant term and the linear and quadratic terms of a specific frequency (ie  cos(wx) ,  cos(wx)sin(wy) , etc). The neurons clearly cluster according to the key frequencies.`
                1. `There's a separate cluster of neurons that are (almost) always active, which contain linear and quadratic terms for a mix of frequencies (ReLU acts as the identity on these neurons, so there's no need to privilege the neuron basis and cluster frequencies).`
                1. ok, so...plots activations of specific neurons as square funcs of the inputs; first in native basis, second in fourier basis
                    1. (we _could_ do....some plots; not simply because our inputs are bigger, but to an extent.)
                    1. (......ok, mm, I have an intuition that activations ought to depend on some inputs but not others; we could....fix a neuron, then systematically vary various inputs to see if the activation varies significantly on some but not others?)
                    1. (& compute that variation via....what?)
                    1. (...& then some will fucking vary nontrivially, and maybe even periodically in 2d but not in a way that's visible if we average out one of them.....or, like, same in 17-d, and what do we do then.)
                    1. (I think we again need some tooling to _locate important circuits_, which'd backprop into locating inputs important to this particular circuit.)
        1. ....ok, let's skip other things & jump to Analysis During Training; that sounds more important & more exciting.
            1. 