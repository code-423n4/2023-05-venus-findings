Isolated lending pools are implemented in a way that those pools are not related in any way.
Each lending pool has its own comptroller and a set of vTokens.

And `Shortfall`, `RiskFund` contracts also handle each lending pool separately according to given `comptroller` parameter.

This is good for isolation, those pools are completely isolated.
So, what's the need for extracting Shortfall and RiskFund functionalities from each lending pool and aggregate them in a single Shortfall, RiskFund contract respectively?