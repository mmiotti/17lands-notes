# Digging into the "Number of Cards Drawn bias" and its impact on GD, GND, and IWD metrics

It appears to me that the GD, GND, and IWD metrics on 17Lands are confounded in a way that 1) favors cards that go in archetypes that prefer the game to go long; and 2) favors cards within each archetype that are better when the games to long. The source of the bias is that win rates of cards can be (and often are) correlated with how many cards are being drawn in a given game.

Some of this bias is addressed in the recent article (https://www.17lands.com/blog/using_wr_data), but I feel like the full impact was missed. The article mentions 1) the potential impact on "WIH WR", 2) the issue of a card being correlated with / co-dependent on other cards in the deck (e.g. a Koma deck being built in a way that favors finding and casting Koma), and a few confounding factors (sub-archetypes, player skill, etc).

However, I believe there is an important bias in the GD, GND, and IWD metrics that exists even when controlling for archetype quality, player skill, and correlation/co-dependence of different cards. The purpose of this write-up is to add to the discussion re: potential issues and caveats with respect to interpreting 17Lands data, and, if I haven't made a mistake or missed something, possibly prompt a follow-up article to illustrate this issue in more detail.

To illustrate what I mean, I constructed an example. This example will show how we can arrive at a table like this:

| Card | WR GD | WR GND | IWD     |
|----- | ----- | ------ | ------- |
| 1    | 33.3% | 66.6%  | -33.3pp |
| 2    | 66.6% | 33.3%  | 33.3pp  |

Even though we might intuitively expect a table like this:

| Card | WR GD | WR GND | IWD     |
|----- | ----- | ------ | ------- |
| 1    | 50.0% | 50.0%  | 0.0pp   |
| 2    | 50.0% | 50.0%  | 0.0pp   |


## Example

**Assumption 1.** Let's say we have a format where only 2 archetypes exist: `Archetype A` (aggro) and `Archetye B` (control). Let's say that the win rate of each of these archetypes is highly correlated with how long the games go, reflected by the total number of cards drawn by the player during each game.

| Length of game (number of cards drawn) | WR Archetype A | WR Archetype B |
| -------------------------------------- | -------------- | -------------- |
| 10                                     | 100%           | 0%             |
| 20                                     | 50%            | 50%            |
| 30                                     | 0%             | 100%           |


**Assumption 2.** Now, let's say we have two cards. `Card 1` only appears in `Archetype A`, `Card 2` only appears in `Archetype B`. Each card is exactly neutral, that is, each card has no effect on the win rate of the corresponding deck, on average, no matter whether it is drawn or not, and each card's impact on the win rate is in no way correlated with other cards that may exist in the deck.

| Card | Only appears in archetype | True impact on WR when drawn vs not drawn |
| ---- | ------------------------- | ----------------------------------------- |
| 1    | A                         | 0pp                                       |
| 2    | B                         | 0pp                                       |


**Asumption 3.** Finally, let's also assume that each of the game durations listed in the first table is equally likely (and, for simplication, that all games result in exactly 10, 20, or 30 cards drawn in total):

| Length of game (number of cards drawn) | Fraction of games that have this length |
| -------------------------------------- | --------------------------------------- |
| 10                                     | 33.3%                                   |
| 20                                     | 33.3%                                   |
| 30                                     | 33.3%                                   |

This final assumption implies that Archetypes A and B both win 50% of games:

```
WR_ArchetypeA = 0.333 * 1.0 + 0.333 * 0.5 + 0.333 * 0.0 = 50%
WR_ArchetypeB = 0.333 * 0.0 + 0.333 * 0.5 + 0.333 * 1.0 = 50%
```

Since neither cards 1 and 2 affect the win rate of their respective decks when drawn, intuitively, we might expect cards 1 and 2 to have a 50% WR when drawn and 50% win rate when not drawn, and an IWD of 0pp. But as you may already sense and I will show, that's not what happens. The key reason is the following table, illustrating the relationship between the duration of a game and the probability to draw a given individual card in the deck:

| Length of game (number of cards drawn) | Chance to draw any given individual card |
| -------------------------------------- | ---------------------------------------- |
| 10                                     | 25%                                      |
| 20                                     | 50%                                      |
| 30                                     | 75%                                      |

Therefore, for Card 1 in archetype A, *we are more likely to draw the card in games we lose than in games we win* (simply because we draw more cards overall, even though the card itself has no effect expected probability to win the game). For Card 2 in archetype B, we are more likely to draw the card in games we win than in games we lose. To put it differently: the subsets of the overall data that reflect the "GD" and "GND" metrics are not random samples of the combined pool of games.

Let's list all possible combinations of game duration, win (which deck / archetype wins), and whether a card is drawn or not, proportional to how often each outcome occurs:

| Game # | Length of game (number of cards drawn) | Win (archetype) | Card is drawn? |
|------- | -------------------------------------- | --------------- | -------------- |
| 1      | 10                                     | A               | Y              |
| 2      | 10                                     | A               | N              |
| 3      | 10                                     | A               | N              |
| 4      | 10                                     | A               | N              |
| 5      | 20                                     | A               | N              |
| 6      | 20                                     | B               | Y              |
| 7      | 20                                     | A               | Y              |
| 8      | 20                                     | B               | N              |
| 9      | 30                                     | B               | Y              |
| 10     | 30                                     | B               | Y              |
| 11     | 30                                     | B               | Y              |
| 12     | 30                                     | B               | Y              |


As we can see, **even though both archetypes have a 50% average win rate, and both cards are neutral (have no effect on the game in terms of expected WR) in both archetypes, one the two cards show vastly different metrics**:

| Card | WR GD | WR GND | IWD     |
|----- | ----- | ------ | ------- |
| 1    | 33.3% | 66.6%  | -33.3pp |
| 2    | 66.6% | 33.3%  | 33.3pp  |

Even though both archetypes A and B have the same overall winrate, and even though both Cards 1 and 2 are exactly average cards within those archetypes, one would conclude from looking at above table that card 2 is vastly superior to card 1.


## Discussion

Based on this observation, we can conclude that the WR GD, WR GND, and IWD metrics favor cards that belong to archetypes that prefer the game to go long (e.g. Quandrix vs Silverquill), and cards within a given archetype that prefer the game to go long (e.g. Rise of Extus vs Killian in Silverquill).

Unless I'm missing or misinterpreting something, this biased is currently not really addressed on the 17Lands website, neither in the caveats of the metrics definition section (https://www.17lands.com/metrics_definitions) nor the recent *Using Win Rate Data* article (https://www.17lands.com/blog/using_wr_data).

The `hint` that gives this away in above table is the WR GND. On 17Lands, however, the "WR GND" column is founded by the overall winrate of the corresponding archetype, but also by the typical quality of the deck that the card is maindecked in (which is why lessons have such a poor WR GND metric: they tend to be maindecked only in pretty bad decks). This makes it trickier to use the "WR GND" column to detect this bias, but comparing the "WR GND" rating to the overall true win % of a given archetype can still help.

The solution is to look at win rate when in deck. Gives a better representation of the average effect of a card across thousands of games. However, strongly confounded by average win rate of an archetype.

Second, it appears that. He writes: *"Historically, these decks also sometimes lose rapidly - think of a multicolor deck splashing bombs that gets run over by a fast aggro deck before it can establish its mana base. Rapid losses and long wins means that in games lost, those cards are much less likely to have been drawn because they were way shorter, while in games won they are more likely to be drawn. This disparity inflates these cards’ GIH WR."* The issue described here is essentially a summary of the underlying bias I'm describing here. However, the article implies that it only affects GIH WR, when it also affects GD, GND, and IWD. The article also im

The currently is not metric that avoids above biases but also is not confounded by average win rate of different archetypes.

## An Example From The Real Data

The following examples from the 17Lands tables (STX, BO1, Premier, as of June 9th 2021) illustrate this:

