XUSD:

Automated Stability Control in Decentralized Finance

Fishy

October 2022


# Table of Contents




# Abstract {#abstract}

XUSD is a stablecoin designed to keep a tight peg to USD using only decentralized collateral. XUSD does this by moving the system price of the stablecoin to correct market deviations from the intended value of $1.


### Explanation of Vault System {#explanation-of-vault-system}

An overcollateralized vault mechanism similar to MakerDAO or Liquity is used to issue XUSD. XUSD is only issued upon borrowing, which anyone can do by depositing collateral. To reclaim their collateral, a vault owner must repay the quantity of XUSD they have borrowed. Each vault has a minimum collateral ratio, a factor that ensures that each vault’s collateral is worth more than its debt. A collateral ratio is the value of a vault’s collateral compared to the value of its debt, expressed in a percentage (e.g. 150%, 120%). If a vault’s collateral ratio falls below the minimum, the vault is marked for liquidation. This ensures that the vault’s collateral is always worth more than its debt. Different liquidation mechanisms have different benefits and downsides.



Liquidation is an important component of any overcollateralized vault system. Liquidation is the process of repaying debts to ensure that the value of a unit of credit (a stablecoin) maintains its value. The ideal liquidation mechanism is instant, slippage-free, scalable and accessible. Liquidation mechanisms trade different characteristics to build a solution that works best for the credit system at hand.

Time

Time introduces a few variables to the liquidation process. More time means cheaper liquidation costs (both more competition as additional bidders join, and more time for the market to correct itself if selling through a TWAMM) and more accessibility (as a long liquidation may be accessible to both manual users and bots). Simultaneously a long liquidation process can increase price risk (as collateral price changes during the process).


## Liquidation Mechanisms {#liquidation-mechanisms}


### MakerDAO v1 {#makerdao-v1}

Any entity may pay off a vault’s debt and receive a proportional amount of collateral plus a bonus known as a liquidation fee (e.g. 10%).

Pros:



* Simple to build

Cons:



* Price-inefficient, small liquidators cannot participate as no partial liquidations


### MakerDAO Auction {#makerdao-auction}

When collateral is marked for auction, any entity may place an on-chain bid to purchase the full collateral. The entity that places the lowest bid (in DAI) purchases the collateral.

Pros:



* Efficient in reducing liquidation costs as liquidator competition drives the price down

Cons:



* Requires one entity to purchase the entire collateral, which can be prohibitive if there are not high-capital liquidators.
* Bars small liquidators from participating (as small liquidators can repeatedly partial liquidate → sell for stablecoins → repeat in a partial liquidation system)


### Interest Protocol (Partial Liquidation) {#interest-protocol-partial-liquidation}

Any entity may pay off a portion of a vault’s debt and receive a proportional amount of collateral plus a bonus known as a liquidation fee (e.g. 10%).

Pros:



* Allows small liquidators to liquidate large positions given strong liquidity for the collateral asset and stablecoin.

Cons:



* Has to use a flat fee, so liquidation is less price efficient than an auction.


### Liquity Stability Pool {#liquity-stability-pool}

Positions are liquidated by draining stablecoins from a smart contract which anyone can deposit to. The liquidation fee profits go to the stability pool participants and a small hybrid (flat + proportional) fee is given to the person who calls the liquidation.

Pros:



* Allows non-technical users to reap the profits of liquidations
* Stability pool depositors protect normal stablecoin holders from bad debt

Cons:



* Situations exist where stability pool depositors are guaranteed to lose money, for example when a vault is liquidated but the collateral is worth less than the debt. This makes the stability fee vulnerable to MEV (quick withdrawal frontrunning the debt absorption onto passive LPs).


### Rising Price Liquidation Auction (XUSD’s liquidation mechanism) {#rising-price-liquidation-auction-xusd’s-liquidation-mechanism}

Debts are denominated in USD (instead of a quantity of stablecoins) for the purpose of liquidations. When liquidation is first triggered, stablecoins are considered to be worth significantly below their peg value for the purpose of paying off the debt. Throughout the auction, the “value” of the stablecoin increases (e.g. at block 1 in the auction the system allows 1 unit to repay $0.80 and at block 50 the same unit is “valued” at $1.00. The “liquidation fee” is 0, as the true “liquidation fee” rises throughout the auction as more collateral is able to be seized by less XUSD. Partial liquidations are supported as .

Pros:



* Partial liquidations allow users of any size to participate
* The “net liquidation fee” starts negative and rises towards infinity, so the price efficiency of an auction is maintained while ensuring that debt can still be recovered in poor market conditions
* Being liquidated can be profitable, as a negative liquidation fee can still be the best execution rate for Stablecoin->ETH on the market.

Cons:



* May be complex to code


### Hollander Liquidation {#hollander-liquidation}

Hollander is dutch auction system which allows participants to submit orders for a portion of the funds on the other side.[^1] It can be modified to support XUSD liquidations. Using Hollander liquidations, a vault’s entire collateral is liquidated, however extra XUSD will be left for the vault owner based on how efficient the dutch auction is.

Pros:



* Dutch auction finds price optimally
* Partial liquidations supported

Cons:



* Dutch Auction mechanic means that liquidation fees are theoretically infinite
* Reduces directional exposure when liquidating


## Peg Problems {#peg-problems}


### The Overpeg Problem {#the-overpeg-problem}

Stablecoins going overpeg is a problem as borrowers who wish to repay their debts must purchase the stablecoin above the target price. This makes borrowing less attractive as borrowers should ideally be able to close their position at any time without paying more than the value that they borrowed.


### Liquidation-based {#liquidation-based}

Liquidation fees temporarily increase the value of a stablecoin beyond the intended value. For example, if a liquidation can be conducted and a liquidator can receive $1.09 per XUSD, the inherent value of the stablecoin at that moment is higher than $1 (specifically liquidation profits – fees to sell collateral). As the inherent value of the stablecoin is higher than the market price, arbitrageurs will correct the market.


### Demand > Supply Problem {#demand->-supply-problem}

If the demand to hold a stablecoin is significantly higher than the amount of users willing to borrow, the stablecoin will go overpeg as holders will require a premium above the target price to sell.

Solutions to the liquidation overpeg problem:

Reducing liquidation fees: By reducing the profit inherent to liquidations, the market price will correct to the target price. This has the downside of making liquidations less likely to happen, as there is less incentive to perform a liquidation and take on the risk of temporarily holding collateral. MakerDAO auctions and the Liquity Stability Pool are both examples of this approach.

1:1 Fungibility with the pegged asset: MakerDAO has accepted USDC 1:1 for DAI, allowing an increase in supply without requiring borrowing. This allows for price-efficient liquidations as liquidators can mint directly at the inherent value.

Solutions to the Demand > Supply Problem:

Floating price: RAI has adopted a floating-price model which abandons the peg in exchange for a perfect balance of supply and demand by changing the system price as the system deviates from peg. RAI is not hard-pegged, as the target price changes. In times of high demand, RAI’s price rate (the rate at which the system price decreases per year) moves downward, disincentivizing holding in favor of borrowing. This allows RAI to reach a market equilibrium as the system balances supply and demand.

1:1 Fungibility with the pegged asset: MakerDAO accepts USDC 1:1 for DAI, allowing an increase in supply without borrowing. This allows buyers of DAI to purchase with USDC instead of having to purchase from someone who has opened a vault. This ensures that buyers will never have to purchase overpeg as infinite USDC is available 1:1 for USD.


### Case Study: Liquity {#case-study-liquity}



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


Data from CoinMarketCap[^2]

Liquity’s LUSD has frequently gone above peg due to the demand>supply problem. Demand to hold LUSD is high due to its reputation for being highly decentralized. LUSD’s one time borrowing fee of 0.5% combined with a 0% interest rate makes borrowing desirable, but the demand to hold LUSD has been historically higher than the demand to borrow. This has caused the price of LUSD to frequently move above $1 as the demand > supply. This move above peg forces borrowers to overpay to close their positions. LUSD holders can functionally hold borrowers hostage, as refusing to sell means that new issuance is the only way to repay. However, LUSD holders frequently break rank and sell, preventing the price from reaching its cap of $1.10 (this cap is informed by the 110% minimum collateral ratio of the system). LUSD cannot go above $1.10 as if it did new borrowers would be strongly incentivized to deposit collateral at an 110% ratio, borrow, and sell their LUSD above $1.10, making a profit as their liquidated collateral is worth less than their borrowed LUSD. The primary issue with the Liquity system is that there is no mechanism to bring demand and supply to equilibrium.


### Case Study: RAI {#case-study-rai}


### 

<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


This chart showing RAI’s market vs redemption price (wut and par respectively) indicates that par price has a definite effect on the value of a stablecoin. However, the reason why par price has an effect is of critical importance. The reason why RAI is able to enforce par = wut is because ultimately those who stand against the system will be liquidated if they continuously do so. Imagine a state where the market was predominantly short RAI, and RAI needed to be removed from circulation to bring the par and market price close to each other. The par would continuously increase, causing eventual liquidation of short positions, balancing demand. 

Conversely, if the market is predominantly long RAI, the system will allow more RAI to be borrowed per ETH. However, this does not reduce prices.


# An Alternative {#an-alternative}

We view the current solutions to the demand > supply problem as insufficient, as both a hard peg and centralization-resistant collateral are critical to a decentralized stablecoin.

The inherent value of a stablecoin comes from its ability to repay debt, which can be reduced or increased by changing [par](#bookmark=id.de757lahfuby). <span style="text-decoration:underline;">In XUSD, debt is denominated in USD instead of a quantity of XUSD. This allows the system to incentivize buying and selling of the stablecoin when under/overpeg respectively.</span>

When [wut](#bookmark=id.6myzh65fxr3x) is higher than the target price, par is reduced, encouraging wut to drop. Conversely, when wut is lower than the target price, par can be increased to encourage wut to rise.

Nikolai Mushegian writes about the moving system price concept: “In 2016 a few early Maker supporters wrote a post about[ modeling the Dai controller](https://steemit.com/makerdao/@kennyrowe/digital-money-a-simulation-of-the-deflation-rate-adjustment-mechanism-of-the-dai-stablecoin), when Dai still had a variable par price controlled in a way similar to Rai is now. While that blog post is useful for a general intuition about how a moving par price incentivizes traders, I think we made a big error in that model.

In that model, we assume that there was a greater incentive to trade toward the par price when there was a greater price deviation. Now I think this is circular reasoning. This makes sense as secondary incentive, when you expect other people to trade towards that price as well. But there needs to be a primary incentive, one that _fundamentally_ presents a profit motive to trade that way, otherwise this reasoning is circular.

The primary incentive are the strategies that generate continuous cashflow by performing rate arbitrage… The first is a simple long rai strategy, used when way > 1. The yield on this strategy is simply way, and it is competitive only when way is greater than the market rate for lending USD in similarly overcollateralized system… The second is a long eth / short rai strategy used when way &lt; 1 (actually, where way &lt; 1.x where x is some value less than market rates for borrowing USD against ETH, plus the fee, the system’s quantity rate)… The third is an eth-neutral strategy that can be used when way &lt; 1 (again, actually when way &lt; 1.x, for some market rate). This is less capital-efficient, meaning the net yield is lower.”[^3]

Just like RAI, XUSD’s “primary incentive” is based on these same three strategies; long XUSD, long ETH/short XUSD, short XUSD. However, XUSD maintains a fiat peg, which we believe will strongly increase its desirability.

These three strategies help XUSD stay at peg:

Peg Incentives

<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")



## Methods of System Price Change {#methods-of-system-price-change}

Now that we’ve established a changing par, we also have to establish how it will change. The rate at which par changes influences various factors such as volatility and the frequency XUSD crosses the target price. There are multiple models for changing par.


### Tick {#tick}

The tick model is the simplest model for changing the par. It increases/decreases the par based on a flat number every block (e.g. 1 basis point).

Pros



* Consistent
* Predictable 

Cons



* Slow to correct large deviations from target price


### Proportional Error (P) {#proportional-error-p}

The error model determines the difference between wut and target price, and uses this to inform how quickly the par will change. For example, the par change per block can be informed by 

<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: equation: use MathJax/LaTeX if your publishing platform supports it. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>

_,_ so that greater deviations from the target price produce a quicker correction. Single Collateral DAI used this model.[^4]<sup> </sup>

Pros



* Quick at correcting large deviations

Cons



* Unpredictable


### Integral Error (I) {#integral-error-i}

“The integral term accumulates error over time, increasing the redemption rate with the duration of error, and not just its magnitude (unlike the P-term, which has no time component)… The I-term is more important for correcting prolonged price deviations — called _steady state error — _as it will ramp up slowly, getting stronger so long as the error persists, and then only resetting slowly over time. The idea is for the I term [to] hold the redemption rate at whatever value brings the error to zero. In effect, the I term is searching for the average long term redemption rate that will keep the error at zero.”-Ameen Soleimani.[^5]

The tick model ensures that system price movement is predictable, so XUSD will not be utilizing P or I terms.


## Supplying a Market Price Feed {#supplying-a-market-price-feed}

Core to XUSD is the ability to have a feed that accurately tells the system wut in a decentralized, non-manipulable way. Since XUSD has no governance, there are no funds available to bootstrap a DAO-owned LP. However, alternative methods can be utilized to incentivize LPs. We can utilize Euler Finance’s Oracle Assessment tool[^6] to determine the safety of the ETH-XUSD Uni V3 LP. We only have to worry about attacks within a few blocks as arbitrage can correct longer attacks.


### Incentivizing LP via low rates {#incentivizing-lp-via-low-rates}

ETH-XUSD LP can be used as collateral for the protocol, and receives a lower [fee](#bookmark=id.56nb4g911gt2) than ETH collateral alone.

Pros



* No system tax required (tax is defined as an arbitrary system addition to fund ETH-XUSD LP)

Cons



* LP is less sticky as demand to LP changes
* LP tokens are not financially desirable, so this mechanism only strengthens the oracle in a minor way (could be removed for simplicity)

	


### Protocol-owned LP via fee income {#protocol-owned-lp-via-fee-income}

ETH-XUSD LP can be acquired by the system as a positive fee can be used to slowly purchase ETH-XUSD LP over time.

Pros



* Protocol-owned LP ensures liquidity is sticky

Cons



* ETH price decreasing reduces oracle reliability as the USD cost to manipulate becomes lower

XUSD-ETH LP backed Risk Share

Locked XUSD-ETH LP is responsible for insuring XUSD holders. In the case of bad debt accrual XUSD-ETH LP will be auctioned until the bad debt is gone. In exchange, Locked XUSD-ETH LPs will earn XUSD from fee. Locked XUSD-ETH LP has a 14-day withdrawal queue to ensure that X depositors withdraw directly before a bad debt event.

Pros



* Covers Risk Share + XUSD-ETH LP

Cons



* Financially complex

Combining incentivized user-owned LP and protocol-owned LP is likely the most reliable way to ensure that XUSD has deep liquidity. The XUSD borrowing  fee can be used to fund protocol-owned liquidity (POL) while ETH-XUSD LP can be used as collateral with a zero/negative fee. 

Using XUSD-ETH LP as a risk share can help to improve the desirability of holding the LP token, which simultaneously deepens liquidity and protects XUSD holders.

~~Within MakerDAO, borrowers owe a quantity of DAI. They pay interest on the DAI they’ve borrowed via the stability fee, so their yearly interest can be calculated with the standard compound interest formula. As compounding every 12 seconds (current Ethereum blocktime) could be considered continuous, the standard formula for continuous interest can work as well.~~

~~A=P(1+r/n)^nt~~

~~A=debt at end of year~~

~~P=Principal~~

~~r=Stability Fee~~

~~n=blocks per year~~

~~t=years~~

~~Within the Maker System, borrowers owe a quantity of DAI. The quantity of DAI owed/owned is the only way the Maker system can incentivize borrowing/holding DAI. This is done through changing the stability fee and DAI savings rate (for borrowers and holders respectively).~~

~~The Reflexer system introduces a new variable, that of a moving price. Within Maker, DAI’s price is fixed to $1, while within Reflexer RAI’s price floats. This ~~


## Actor Centric Modeling {#actor-centric-modeling}

Models of DAI, RAI, and XUSD are included to show the differences between how the systems issue debt, and the differences actors have interacting with one system vs. another.


### MakerDAO (DAI) {#makerdao-dai}

A borrower deposits 100 ETH (worth $100k) collateral, which they use to borrow 80k DAI. The borrower sells this DAI for 80 ETH to a stablecoin holder. The borrower’s debt increases every block at a rate of 2% APY due to the stability fee, while the stablecoin holder receives 0.01% APY due to the DAI savings rate. The remaining interest is collected by MakerDAO. As DAI is redeemable 1:1 with USDC, any price deviation above or below $1 will be closed by an arbitrageur, who can redeem or acquire USDC 1:1 for USD. At the end of the year, DAI is valued at $1.

<p id="gdcalert5" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert6">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")


Maker is quantity focused - can only use quantity lever, hard peg


### Reflexer (RAI) {#reflexer-rai}

A borrower deposits 100 ETH (worth $100k) collateral, which they use to borrow 20k RAI (priced at the time of borrowing at $4). The borrower sells this RAI for 80 ETH to a stablecoin holder. The quantity of the borrower's debt increases every block at a rate of 2% APY due to fee, while the value of their debt decreases every block at a rate of -5% APY due to [way](#bookmark=id.uaern3423xcv). RAI decreases in USD value as way is negative. At the end of the year, the borrower owes 20,400 RAI, which is worth $77,520. The borrower earned $2,480, even though they owe 400 more RAI. At the end of the year, RAI is valued at $3.80. The goal of way in RAI is to drag RAI’s wut and par closer to each other (not to any specific value).

<p id="gdcalert6" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image5.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert7">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image5.png "image_tooltip")


Reflexer can use both quantity and “quality” (price) - can use way and fee, no peg


### XUSD {#xusd}

A borrower deposits 100 ETH (worth $100k) collateral, which they use to borrow 80,000 XUSD (wut and par initially = $1). The borrower sells this XUSD for 80 ETH to a stablecoin holder. The quantity of the borrower's debt increases every block at a rate of 2% APY due to fee, while the system value of their debt changes every block due to way. If wut > $1, par increases by 1 bps per block. If wut &lt;$1, par decreases by 1 bps per block. Assume at the end of the year wut = $1 and par = $0.95. This is a realistic case, as the Liquity case study shows that par value is not always 1:1 with wut value. In this case, the borrower’s debt of 102 XUSD is valued at $96.9 by the system, while the XUSD they sold is valued at $100 by the market. At the end of the year, XUSD is valued at $1. The goal of way in XUSD is to drag XUSD’s wut to $1. XUSD’s target wut value will never change from $1.

<p id="gdcalert7" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image6.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert8">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image6.png "image_tooltip")


XUSD can use both quantity and price - can use way and fee, hard peg to $1.

The core difference between RAI and XUSD is the “target” price. RAI’s target wut changes over time, while XUSD’s will always be $1. 


## Recovery Mode {#recovery-mode}

Liquity’s recovery mode is a unique system built to ensure each LUSD is backed by at least $1.[^7]

XUSD’s system maintains two separate collateral ratios, the ICR (Individual Collateral Ratio) and the TCR (Total Collateral Ratio). The ICR is an individual vault’s collateral ratio, while the TCR is the total collateral ratio of the XUSD system. The ICR is determined using par price for the value of XUSD, while the TCR uses the target wut. The TCR ensures that no matter how far par drops, 1 XUSD will be backed by $1 of collateral. The TCR is calculated with (total value of all collateral/quantity of issued XUSD. For example, if the system had $150M of collateral and $100M of issued XUSD, the TCR would be 1.5, or 150%.


### Recovery Mode A {#recovery-mode-a}

If the TCR falls below 120%, the system discovers how much XUSD needs to be repaid to reach 120%. For example, if the TCR is 110% with $110M of collateral and 100M issued XUSD, then ~8.33M XUSD needs to be reclaimed. Then, the system triggers liquidations starting from the lowest ICR until 8.33M XUSD are repaid. Assuming an average liquidation fee of 10% (particularly high considering XUSD’s liquidation mechanism), there is now 100,833,333 of collateral and 91.67M of debt. With a liquidation fee of 10%, the collateral ratio stays exactly the same at 110%. However, lower liquidation fees made possible by XUSD’s rising price liquidation auction ensure that most liquidations will increase the TCR of the system.


### Recovery Mode B {#recovery-mode-b}

If the TCR falls below 120%, all vaults below 120% ICR are marked for liquidation.


## Capped/Minimum Borrowing {#capped-minimum-borrowing}

Borrowing caps/minimums help protect the system against positions that are unprofitable to liquidate. There are two main types of “un-liquidatable” positions. The first is a position which is extremely small, in which the gas fees to liquidate > the profit of the liquidation. A minimum borrowing threshold of 2k XUSD can prevent the position from being liquidated, as the 20% max liquidation fee of the rising price liquidation auction yields a minimum of $400, far higher than Ethereum gas fees.

The second is a position which is excessively large, primarily positions where the size is far greater than the depth of liquidity available for the collateral asset. 


## Governance {#governance}


### The Immutability Path {#the-immutability-path}

The XUSD system is immutable and without governance, so no new collateral can be added to the system. However, any person may deploy a new version of the system with any type of collateral. A new stablecoin will be minted along with this system. This allows users to tweak the collateral types, collateral factors, interest rate curves, and so on. This allows users from a wide variety of backgrounds to create a stablecoin which represents their values, and which they are comfortable using. For example, the Compound community may wish to create a stablecoin backed by assets on Compound, while a decentralization-focused community may only want to accept ETH collateral. Permissionless deployment fulfills the needs of both communities.


### The DAO Path {#the-dao-path}

XUSD could use a DAO to manage various aspects of the stablecoin including collateral types, controller parameters, oracle maintenance, and yield direction. Voting rights within the DAO can be based on ownership of a governance token, risk share, or stablecoin. Vitalik Buterin has criticized token voting in [Moving beyond coin voting governance](https://vitalik.ca/general/2021/08/16/voting3.html).

In order to protect XUSD holders from malicious governance proposals, XUSD will have veto rights over any governance proposal. 


#### Threats {#threats}

The undesirable scenarios the XUSD system must protect itself from are: 



* special prioritization of certain shareholders, treating any given XUSD or gov token holder differently (paying certain gov token holders but not others)
* modifying collateral types/factors resulting in reduced XUSD collateralization, excessive volatility, centralization etc.
* Addition of centralized components to the system
* Any change which would cause stablecoin holders to receive less than the intended value of the stablecoin
* Vote buying
* System shutdown without valid reason

Buterin states: “One possible mitigation to the above issues, and one that is to varying extents being tried already, is to put limits on what coin-driven governance can do. There are a few ways to do this:...



* Limit governance to fixed parameter choices: Uniswap does this, as it only allows governance to affect (i) token distribution and (ii) a 0.05% fee in the Uniswap exchange. Another great example is[ RAI's "un-governance" roadmap](https://docs.reflexer.finance/ungovernance/governance-minimization-guide), where governance has control over fewer and fewer features over time.
* Add time delays: a governance decision made at time T only takes effect at eg. T + 90 days. This allows users and applications that consider the decision unacceptable to move to another application (possibly a fork). Compound[ has a time delay mechanism](https://compound.finance/docs/governance) in its governance, but in principle the delay can (and eventually should) be much longer.
* Be more fork-friendly: make it easier for users to quickly coordinate on and execute a fork. This makes the payoff of capturing governance smaller.

The Uniswap case is particularly interesting: it's an intended behavior that the on-chain governance funds teams, which may develop future versions of the Uniswap protocol, but _it's up to users to opt-in_ to upgrading to those versions. This is a hybrid of on-chain and off-chain governance that leaves only a limited role for the on-chain side.

But limited governance is not an acceptable solution by itself; those areas where governance is needed the most (eg. funds distribution for public goods) are themselves among the most vulnerable to attack. Public goods funding is so vulnerable to attack because there is a very direct way for an attacker to profit from bad decisions: they can try to push through a bad decision that sends funds to themselves. Hence, we also need techniques to improve governance itself…”


#### Veto Council {#veto-council}

Buterin states: “Time delays plus elected-specialist governance: this is one possible solution to the ancient conundrum of how to make an crypto-collateralized stablecoin whose locked funds can exceed the value of the profit-taking token without risking governance capture. The stable coin uses a price oracle constructed from the median of values submitted by N (eg. N = 13) elected providers. Coin voting chooses the providers, but it can only cycle out one provider each week. If users notice that coin voting is bringing in untrustworthy price providers, they have N/2 weeks before the stablecoin breaks to switch to a different one.”

This same concept can be applied to all threats which are based on 51% attacks. A council of individuals can be elected to veto harmful governance proposals. Election of individuals reduces 51% attack risk as each individual member of the council has their personal reputation at stake.

Council members should be elected at the DAO’s beginning, and only removed by governance vote. Only one member may be removed per month. This ensures that a 51% attacker must hold their tokens for 

<p id="gdcalert8" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: equation: use MathJax/LaTeX if your publishing platform supports it. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert9">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>

 months in order to prevent any vetoes (assuming initially elected members are honest). Council members should have prominent positions at other cryptocurrency organizations so that their reputation is at stake. Long voting periods and timelocks are default for all system changes to ensure that token holders can exit if they disagree with a proposal, and so that the council has time to veto hostile proposals.

The council could have the ability to reduce an account's holdings of the governance token. This can be used to slash accounts that participate in vote-selling or 51% attacks.

Governance should only be able to perform a limited set of actions, such as changing collateral types/factors, changing interest rates, directing yield, or modifying the price controller.


## Considering {#considering}


### Redemption {#redemption}

Liquity Protocol allows LUSD to be redeemed for ETH, which is desirable as it strengthens the peg floor and increases the liquidity of LUSD.[^8] LUSD can be redeemed for $1 of ETH from the vault with the lowest collateral ratio. Liquity applies a floating fee to this interaction. Liquity’s fee is 

<p id="gdcalert9" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: equation: use MathJax/LaTeX if your publishing platform supports it. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert10">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>

, where baseRate is a variable that increases based on the amount of LUSD being redeemed (proportional to the total LUSD supply), but decreases over time. baseRate slows down the redemption process as times of high redemption incur higher fees. This partially protects LUSD borrowers from being redeemed against if Liquity’s ETH oracle price &lt; ETH market price. XUSD will be implementing a copy of this mechanism, as it protects against oracle malfunctions while maintaining ETH redeemability.


### USD Denomination vs XUSD Denomination {#usd-denomination-vs-xusd-denomination}

~~USD Denominated -> total price of debt cannot change, quantity of debt can~~

~~If above peg and borrowing - sell to buy back when below or at peg~~

~~If below peg and borrowing - buy below peg, repay debt at or above~~

~~Any vault owner can repay at any time for the exact USD value of their borrow~~

XUSD denominated -> total price of debt can change, quantity of debt cannot

If above peg and borrowing - sell to buy back when below or at peg

If below peg and borrowing - buy below peg, repay debt at or above

Any vault owner can repay at any time for the exact XUSD quantity of their borrow


## Attacks {#attacks}


### Par devaluation attack {#par-devaluation-attack}

If no Total Collateral Ratio existed in the system, the par could potentially fall to a point where each XUSD is backed by less than $1 in the system. This could be caused if XUSD holders refused to sell and caused the par to decrease so low that more XUSD could be minted than the dollar value of collateral in the system.


### Minimum Par Squeeze {#minimum-par-squeeze}

However, the TCR introduces another problem. If the system falls to the minimum acceptable par value, the market price could still be over-peg. Therefore, more levers need to be pulled to reduce the desirability to hold XUSD.

Nikolai Mushegian states: “In a naive implementation, the amount of debt owed exceeds the amount of credits in the system, which combined with lack of `way` for setting a negative effective rate, would cause the system to inevitably collapse, when interest owed is greater than what the system can actually pay.”

Since the amount of debt constantly increases and the credit supply is constant, the debt will eventually increase beyond the ability for the system to pay back. To solve this, new XUSD is minted and traded for risk shares, equalizing the amount of debt and credit. Unfortunately, this still does not equalize the demand and supply.

A tax on XUSD holders (which distributes XUSD to risk share holders) could serve to help reduce the demand for XUSD (as the quantity of XUSD holders have decreases).

How to incentivize holding

Increase price (p rate)

Increase quantity holders have (q rate)

Endanger borrowers collateral ratio (increase par) - increasing demand to repay 

How to disincentivize holding

Take units of XUSD from holders when par is at minimum and overpeg

How to incentivize borrowing

Decrease price (p rate) - TCR limits the maximum decrease

Decrease quantity owed (q rate) - TCR limits the maximum amount that can be minted

Make collateral ratio healthier (decrease par) - unsure of the degree this helps


## Glossary {#glossary}

System price (par) – the price a vault considers 1 unit of XUSD to be worth. This informs the collateral ratio of a position, and is used to determine how much XUSD can be borrowed.

Price rate (way)  - the rate at which the par of XUSD moves annually, expressed similar to a traditional interest rate such as -3%, or +3%. 

Quantity rate (fee) - the “traditional interest rate”, or the rate at which the quantity of XUSD owed changes

Market price (wut)– the market price of XUSD

Target price – the wut the system is trying to reach ($1 in XUSD’s case).

Todo

Add PGP signature to personal website + end of paper

Find a real name for XUSD

Formula + term for max par decrease given X level of current collateralization, min ICR, and min TCR

	


<!-- Footnotes themselves at the bottom. -->
## Notes

[^1]:
    <sup> https://github.com/Destiner/hollander-core</sup>

[^2]:
     https://coinmarketcap.com/currencies/liquity-usd/

[^3]:
    <sup> </sup>https://bank.dev/implied-yield.html

[^4]:
    https://steemit.com/makerdao/@kennyrowe/digital-money-a-simulation-of-the-deflation-rate-adjustment-mechanism-of-the-dai-stablecoin

[^5]:
    <sup> </sup>https://medium.com/reflexer-labs/rai-i-term-tuning-update-a6a8de72f86

[^6]:
    <sup> <a href="https://oracle.euler.finance/">https://oracle.euler.finance/</a></sup>

[^7]:
    <sup> https://docs.liquity.org/faq/recovery-mode</sup>

[^8]:
     https://docs.liquity.org/faq/lusd-redemptions
