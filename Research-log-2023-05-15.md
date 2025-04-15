# 2023-05-15

1. Babble! How what we know about 1-layer TSs now fits into digit-prediction!
    1. A: .....Iiiii think it doesn't. At all.
    1. we have an understanding of skip-3grams, right. but the thing is that our `b` is always `=`, & so, entirely uninformative! or rather—if we were looking for an optimal 1-layer TS, we'd look for....a thing that optimizes skip-3grams whose `b` is `=` and `c` is the output digit.
    1. & that sounds.....if not impossible, at least hard. For one, naive 1-layer TS is blind to digit positions before it!! that's fixed by adding positional encoding, & I actually dunno how that works technically, at all! Thaat's a problem.
    1. but.....ok, suppose....the position-agnosticism problem is solved _somehow_, and that the token dictionary _effectively_ has not only token but its position encoded.
    1. in that case....we still don't have enough info, I don't think? for the 2nd digit, we can learn to attend to `x3` and `y9` tokens (there's always exactly one of both in the context), but.....what's an optimal combination of (=,x) & (=,y) attention and logits that x & y induce on `c`?
    1. I feel like...it shouln't be possible to do better than a flat distribution again, because...in isolation x & y predict nothing about `c`? and.....I think....it shouldn't be possible to do better than isolated prediction error by ≈linearly mixing x's & y's prediction?
    1. buuuut I maybe should formalize this smh.
