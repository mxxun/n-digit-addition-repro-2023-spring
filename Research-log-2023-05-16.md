# 2023-05-16

1. ok! what did we do yesterday.
    1. we meditated on how...plain 1-layer transformer (+ softmax) feels like it Cannot predict a mixed function of 2 previous tokens better than either of the tokens can
    1. --this phrasing might not be exactly correct, but it's close; it's true if output distr of Z is flat if conditioned on any single value of X or Y
    1. a sketch of the proof is basically:
        1. the output equation is: Softmax^i(αV^X(x,i) + (1-α)V^Y(y,i)) (i-dim vector op, here)
        1. First, α does not matter: if it's 0 or 1 the thing collapses, if it's not we can think about W^X = 1/αV^X and corresponding W^Y.
        1. Second: intuit that V^X(x,i) are basically...logits; if we fix x and y, and V^X(x,i) = log a_i and V^X(y,i) = log b_i then Z(0) : Z(1) : Z(i) = a_0×b_0 : a_1 × b_1 : a_i×b_i.
        1. correspondingly, the action "change V^X(x) around" means basically "multiply those a_j in w/e direction we want", shunting odds of Z(i) at _this_ x but _all_ y.
        1. Third, intuition (hypothesis): our loss is such a function that..the optimal shunting of V^X(x) is such that the correct answer at all y has the same log-odds.
            1. when there're 2 digits, it means that the optimal shunting makes Z(i) at y=0 and y=1 looks like t:1 & 1:t.
            1. that's easy to show directly. let 1-odd be the correct answer and assume we shunted away by s, to t:s & 1:st. compute the loss in both cases.
            1. in symmetric, it's 2log (t+1); in shunted, it's log(t/s + 1) + log (ts+1). compare: exponentiate, multiply.
            1. (t+1)² Vs (t/s+1)(ts+1) => 2t Vs ts + t/s => 2 Vs s+1/s. s+1/s has a minimum in 2, when s is exatly 1, so, yeah, symmetric odds are optimal.
            1. (handwavingly extrapolating to n digits: I think this is true for odds between the correct digit & any given other one. ...Iunno if that's enough to constrain the optimal prob-vector to a single value, but..probably enough to constrain it to "odds of the right i are the same across y")
        1. Third & a half, this constrains the whole tensor of x,y,i-odds quite a lot! in 2-digit case, it has exactly 1 free parameter: half of cells have odd 1 & the other, odd `t`.
        1. Fourth, _actually_ there's no free parameter & t = 1. that's because all shunting preserves some invariants on cells' odds products — a product of odds of `t`-cells in particular — and given that this invariant _can_ have value 1 (given the flat distr), it therefore must have that value always.
    1. (TBC I don't have a good idea how to extrapolate this into uneven distributions, or into a case where Z(x,y) is actually a distr instead of a point.)
        1. (I think the same approach will work! Just need to figure where the optimum sits if the distr is uneven.)

1. ..ok, done I suppose. now what.
    1. we...could continue reading amfftc paper? that....sounds like Why Not, yeah, let's chew on it some.
    1. ---we could also try to meditate on......how a 1-layer Attn _could_ approximate at least the collapsed-to-log2 distr?
        1. ---actually, the result above implies it....can't...? & therefore I'm confused as to what _happens_ when it manages to collapse to log2 anyway. (I think it does, eventually? yeah; see [[Research-log-2023-05-05]] results)
    1. Or how a 1-layer Attn+MLP could do....that.
        1. --_that_ sounds like it should be done _after_ reading some additional papers about interpeting MLP layers.
1. ok, fine, let's go amfftc.