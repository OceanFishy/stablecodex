Liquidation Mechanisms
=====

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



* Dutch Auction mechanic means that liquidation fees are theoretically infiniteReduces directional exposure when liquidating

<!-- Footnotes themselves at the bottom. -->
## Notes

[^1]:
    <sup> https://github.com/Destiner/hollander-core</sup>


