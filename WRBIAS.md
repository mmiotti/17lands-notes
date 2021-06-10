# Digging into the "Number of Cards Drawn bias" and its impact on GIH, GND, and IWD metrics

The GIH, GND, and IWD metrics on 17Lands are confounded in a way that 1) favors cards that go in archetypes that prefer the game to go long; and 2) favors cards within each archetype that are better when the games to long. The source of the bias is that win rates of cards can be (and often are) correlated with how many cards are being drawn in a given game. This bias can substantial affect the GIH, GND, and IWD metrics even when controlling for archetype quality, player skill, and correlation/co-dependence of different cards.

Some of this bias is addressed in the recent article (https://www.17lands.com/blog/using_wr_data). In particular, the article mentions the potential impact of a *"multicolor deck splashing bombs"* (that likes to draw cards and the game to go long) on typical GIH WR vs an aggro deck. However, I believe that this bias wasn't explored and highlighted enough. It may be substantially larger than some of the other confounding factors mentioned in the article (that are being given a lot more room). The impact appears to be large enough to matter for comparing e.g. Killian to Rise of Extus or even Silverquill Apprentice in Silverquill, and not just a Killian in aggro Silverquill to a splashed Magma Opus in a controlling Quandrix deck.

The purpose of this write-up is to add to the discussion re: potential issues and caveats with 17Lands metrics, and possibly prompt a follow-up article to illustrate this issue in more detail.

To illustrate what I mean, I constructed an example. This example will show how we can arrive at a table like this:

| Card | WR GIH | WR GND | IWD     |
|----- | ------ | ------ | ------- |
| 1    | 33.3%  | 66.6%  | -33.3pp |
| 2    | 66.6%  | 33.3%  | 33.3pp  |

in a case where we might intuitively expect a table like this:

| Card | WR GIH | WR GND | IWD     |
|----- | ------ | ------ | ------- |
| 1    | 50.0%  | 50.0%  | 0.0pp   |
| 2    | 50.0%  | 50.0%  | 0.0pp   |

even when, as I said, controlling for archetype quality, player skill, and correlation/co-dependence of different cards.


## Example

**Assumption 1.** Let's say we have a format where only 2 archetypes exist: `Archetype A` (aggro) and `Archetye B` (control). Let's say that the win rate of each of these archetypes is highly correlated with how long the games go, reflected by the total number of cards drawn by the player during each game.

| Length of game (number of cards drawn) | WR Archetype A | WR Archetype B |
| -------------------------------------- | -------------- | -------------- |
| 10                                     | 100%           | 0%             |
| 20                                     | 50%            | 50%            |
| 30                                     | 0%             | 100%           |


**Assumption 2.** Now, let's say we have two cards. `Card 1` only appears in `Archetype A`, `Card 2` only appears in `Archetype B`. Each card is exactly neutral, that is, each card has no effect on the win rate of the corresponding deck (neither positive nor negative), on average, neither when drawn or when in deck, and each card's impact on the win rate is in no way correlated with other cards that may exist in the deck.

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

Since neither cards 1 and 2 affect the win rate of their respective decks when drawn, intuitively, we might expect cards 1 and 2 to have a 50% WR when drawn and 50% win rate when not drawn, and an IWD of 0pp. But that's not what happens. The key reason is the following table, illustrating the relationship between the duration of a game and the probability to draw a given individual card in the deck:

| Length of game (number of cards drawn) | Chance to draw any given individual card |
| -------------------------------------- | ---------------------------------------- |
| 10                                     | 25%                                      |
| 20                                     | 50%                                      |
| 30                                     | 75%                                      |

Therefore, for Card 1 in archetype A, *we are more likely to draw the card in games we lose than in games we win* (simply because we draw more cards overall, even though the card itself has no effect expected probability to win the game). For Card 2 in archetype B, we are more likely to draw the card in games we win than in games we lose. To put it differently: the subsets of the overall data that reflect the "GIH" and "GND" metrics are not random samples of the combined pool of games.

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
| 9      | 30                                     | B               | N              |
| 10     | 30                                     | B               | Y              |
| 11     | 30                                     | B               | Y              |
| 12     | 30                                     | B               | Y              |


As we can see, **even though both archetypes have a 50% average win rate, both cards are neutral (have no effect on the game in terms of expected WR) in both archetypes, and neither card's performance is correlated with other cards in the deck, the two cards show vastly different metrics**:

| Card | WR GIH | WR GND | IWD     |
|----- | ------ | ------ | ------- |
| 1    | 33.3%  | 66.6%  | -33.3pp |
| 2    | 66.6%  | 33.3%  | 33.3pp  |

One would conclude from looking at above table that card 2 is vastly superior to card 1, even though both archetypes A and B have the same overall winrate, and even though both Cards 1 and 2 are exactly average cards within those archetypes.


## Discussion / Ways to Avoid Bias

Based on this observation, we can conclude that the WR GIH, WR GND, and IWD metrics favor cards that belong to archetypes that prefer the game to go long (e.g. Quandrix vs Silverquill), and cards within a given archetype that prefer the game to go long (e.g. Rise of Extus vs Killian in Silverquill).

The recent article on using WR data (https://www.17lands.com/blog/using_wr_data) notes *"[...] think of a multicolor deck splashing bombs that gets run over by a fast aggro deck before it can establish its mana base. Rapid losses and long wins means that in games lost, those cards are much less likely to have been drawn because they were way shorter, while in games won they are more likely to be drawn. This disparity inflates these cards’ GIH WR."* This statement gets rather close to the issue discussed here, but it only mentions the impact on GIH WR, and (in my perception) misses the importance of this effect on all cards across the board, not just splashed multicolor bombs.

The `hint` that gives this issue away in above table is the WR GND. On 17Lands, however, the "WR GND" column is confounded by the overall winrate of the corresponding archetype, but also by the typical quality of the deck that the card is maindecked in and the typical quality of the player that player that card (which is why lessons have such a poor WR GND metric, which refers to a case where they are maindecked). This makes it trickier to use the "WR GND" column to detect this bias, but comparing the "WR GND" rating to the overall true win % of a given archetype can still be a start: If the WR GND rating is higher than the average WR of the archetype(s) that the card goes in (and you can assume no significant bias in average deck quality within those archetypes or in player skill), then the card may have an inflated WR GND, deflated WR GIH, and therefore deflated IWD. If it's the other way round (WR GND lower than average WR of archetype(s)), then the card may have deflated WR GND, inflated WR GIH, and therefore inflated IWD.*

<!-- Another approach to look at 17Lands data without encountering this bias would be to look at "win rate when in deck", adjusted for the typical win % of the archetype(s) that the card goes in. Across thousands of games, this should still give an idea of how good the card is, and, conveniently, it will automatically account for biases in terms of what other (weaker) cards you might need to play to make playing the card in question good.
 -->

## An Example From The Real Data

The following examples from the 17Lands tables (STX, BO1, Premier, as of June 9th 2021) illustrate this:

| Card                    | GP WR | WR GIH | WR GND  | IWD    |
| ------------------------| ----- | ------ | ------- | ------ |
| Killian                 | 59.1% | 61.3%  | 56.8%   | 4.6pp  |
| Silverquill Apprentice  | 58.1% | 56.6%  | 59.0%   | -2.4pp |

These two cards have surprisingly similar win rates "when in deck". However, Silverquill Apprentice really shines in the fastest games, whereas Killian is a bit better late game and can also easily be played in more controlling Silverquill decks. This will somewhat inflate the WR GIH for Killian compared to Silverquill Apprentice, and deflate WR GND.

However, this example also illustrates that "GP WR" isn't a perfect metric either: if Silverquill Apprentice is mostly played in the most aggressive Silverquill decks (known for their notoriously high win rates), and Killian is also played in more controlling decks or even splashed in multicolor decks (both of which can probably be expected to have lower typical win rates than Silverquill aggro), then the "true" difference in the quality of the cards lies somewhere in between that the "GP WR" column would suggest and that the "WR GIH" or "IWD" columns would suggest.

