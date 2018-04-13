---
layout: post
author: zhiayang
title:  "Everything is Broken"
date:   2017-12-02 2100 +0800
categories: flax problems
---

Well, turns out the entirety of arrays and shit are completely and utterly broken. There's some madness going on with cloning and growing and reallocations.
Somehow, we have a bug where some particular string's length is being corrupted, but not the rest. I hate this stupid memory debugging, it sucks ass.

Here's the deal:
```
PRE X
POST X
ZERO
ONE
grow (realloc) array: 0000016644113BE8 / len 1 / cap (old) 1 / cap (new) 4
TWO
THREE
FOUR
grow (realloc) array: 00000166447CA508 / len 4 / cap (old) 4 / cap (new) 8
FIVE ALL OK - (aaa, 00000166441907E7, 3), (BBB, 00000166441907F3, 3), (ccc, 00000166441907FF, 3), (DDD, 000001664419080B, -842150451),
(eee, 0000016644190817, 3)
OK 2
OK 3
hello, clone, 0/5
clone string 000001664481CFC0, 00000166441907E7, 3, 12
clone string 'aaa' / 3 / 000001664481CFC8
hello, clone, 0/5
hello, clone, 1/5
clone string 000001664481D7E0, 00000166441907F3, 3, 12
clone string 'BBB' / 3 / 000001664481D7E8
hello, clone, 1/5
hello, clone, 2/5
clone string 000001664481D420, 00000166441907FF, 3, 12
clone string 'ccc' / 3 / 000001664481D428
hello, clone, 2/5
hello, clone, 3/5
clone string 0000000000000000, 000001664419080B, -842150451, -842150442
```

Notice how, for the string 'DDD', the length is some super negative number, but those around it aren't affected at all. I'm not sure why this is
happening, but I'm not looking forward to debugging this mess tomorrow (or today, rather).

