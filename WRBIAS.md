# WR

The fundamental cause of the bias is that **the in rates of the decks that specific cards go in are correlated with how many cards are being drawn in a given game.**

Let's also say that there are only 2 types of games, short ones (10 cards drawn) and long ones (20 cards drawn).

2 Archetypes, Archetype 1 (super aggro) and Archetype 2 (control).

| Length of game (number of cards drawn) | WR Archetype A | WR Archetype B |
| -------------------------------------- | -------------- | -------------- |
| 10                                     | 100%           | 0%             |
| 20                                     | 50%            | 50%            |
| 30                                     | 0%             | 100%           |

Now we have two cards. `Card 1` only appears in `Archetype 1`, `Card 2` only appears in `Archetype 2`.

Let's say each card is exactly neutral, that is, each card has no effect on WR of the corresponding deck, on average, no matter whether it is drawn or not.

Based on all of this, we could expect cards 1 and 2 to have 50% WR when drawn and 50% win rate when not drawn, and a XXX of 0pp.

But, as you may already sense, that's not what happens, because of the shorter a game, the smaller the chance to draw a given individual card:

| Length of game (number of cards drawn) | Chance to draw any given individual card |
| -------------------------------------- | ---------------------------------------- |
| 10                                     | 25%                                      |
| 20                                     | 50%                                      |
| 30                                     | 75%                                      |

Therefore, for Card 1 in Archetype 1, we are more likely to draw the card in games we lose than in games we win.

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
| 1    | 66.6% | 33.3%  | 33.3pp  |

Favors cards that belong to archetypes that prefer the game to go long (e.g. Quandrix vs Silverquill). Favors cards within a given archetype that prefer the game to go long (e.g. Rise of Extus vs Killian in Silverquill).

The solution is to look at win rate when in deck. Gives a better representation of the average effect of a card across thousands of games. However, strongly confounded by average win rate of an archetype.

The currently is not metric that avoids above biases but also is not confounded by average win rate of different archetypes.

Of course, the WR GD and GW GND (and IWD) metrics can still be useful, especially for comparing cards within a given archetype to each other.