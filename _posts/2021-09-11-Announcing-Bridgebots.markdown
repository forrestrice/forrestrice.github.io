---
layout: post
title:  "Announcing Bridgebots"
date:   2021-09-11 12:45:09 -0700
categories: bridge bridgebots
---

Over the past year I have been pursuing a few different bridge programming projects. Eventually I would like to explore the possibility of building ML models which can play and bid competitively, understand a variety of opposing systems, and explain to humans or other bots their own actions and system. To that end I began to collect training data consisting of the bidding and play of high-level players. I quickly encountered two main problems:

1. Bridge data formats such as PBN and LIN are difficult to parse or interact with programmatically.
2. Many of the tools built to consume or produce bridge data are closed source and/or only available on Windows.

Today I am pleased to announce the first release of my solution to both these problems: [Bridgebots](https://github.com/forrestrice/bridge-bots)!

### What is Bridgebots?
I'm so glad you asked. Bridgebots is a [MIT Licensed](https://en.wikipedia.org/wiki/MIT_License) collection of libraries and scripts built to aid in the processing of bridge data. It is implemented in python in order to be portable and approachable by a wide range of programmers. Today I will be going over the functionality of [Bridgebots Core](https://github.com/forrestrice/bridge-bots/tree/master/bridgebots), the primary library for parsing and representing bridge data.

### Getting Started
As of this writing bridgebots core is on version 0.0.6 and is available on [pypi](https://pypi.org/project/bridgebots/). It requires python 3.7 or later. Begin with a simple:

```shell
pip install bridgebots
```

### Building Blocks
In order to process bridge data we need representations for the myriad components that make up the game. Starting with the simplest we have:
* Cardinal Direction
* Card (composed of a Suit and a Rank)
* Bidding Suit (Suits with the addition of No Trump)

Bridgebots uses [Enums](https://docs.python.org/3/library/enum.html) to represent these. 

```python
from bridgebots.deal_enums import BiddingSuit, Direction, Rank, Suit

dealer = Direction.EAST
boss_suit = Suit.SPADES
opener_suit = BiddingSuit.HEARTS
contract_suit = BiddingSuit.NO_TRUMP
highest_rank = Rank.ACE
lowest_rank = Rank.TWO
```
Each comes with helper methods for creation, comparison, or general utility.
#### Creation
```python
# Relies on python 3.8 f-strings
print(f"{Direction.from_str('N')=}")
print(f"{Suit.from_str('C')=}")
print(f"{BiddingSuit.from_str('H')=}")
print(f"{Rank.from_str('K')=}")
```
```console
Direction.from_str('N')=NORTH
Suit.from_str('C')=CLUBS
BiddingSuit.from_str('C')=HEARTS
Rank.from_str('K')=KING
```
#### Comparison
```python
print(f"(Suit.HEARTS > boss_suit): {Suit.HEARTS > boss_suit}")
print(f"(opener_suit > contract_suit): {opener_suit > contract_suit}")
print(f"(highest_rank > lowest_rank): {highest_rank > lowest_rank}")
```
```console
(Suit.HEARTS > boss_suit): False
(opener_suit > contract_suit): False
(highest_rank > lowest_rank): True
```

#### Utility
```python
print(f"{dealer.next()=} {dealer.partner()=} {dealer.previous()=}")
print(f"{dealer.abbreviation()=}, {boss_suit.abbreviation()=}, {Rank.TEN.abbreviation()=}")
```
```console
dealer.next()=SOUTH dealer.partner()=WEST dealer.previous()=NORTH
dealer.abbreviation()='E', boss_suit.abbreviation()='S', Rank.TEN.abbreviation()='T'
```

### Cards, Hands, and Deals, Oh My!
In order to record the bidding and play of a bridge hand we need to combine our building blocks into more complex concepts. Bridgebots represents cards, hands, and deals using [Python Classes](https://docs.python.org/3/tutorial/classes.html). Just like the enums, these classes provide helper methods for a variety of use cases. For this example we'll use a hand from a recent club game:

![board 1](/assets/bridge/bridgebots/announce_board_1.png)

#### Let Me Give You a Hand
```python
from bridgebots.deal import Card, PlayerHand, Deal
from bridgebots.deal_enums import Direction, Rank, Suit

club_nine = Card.from_str("C9")
diamond_three = Card(Suit.DIAMONDS, Rank.THREE)

south_hand = PlayerHand.from_string_lists(
    clubs=["K", "8", "7", "4"], diamonds=["A", "T", "4"], hearts=["K", "7", "2"], spades=["K", "3", "2"]
)
north_card_strings = ["SA", "S5", "HQ", "HT", "H9", "H5", "H4", "DJ", "D3", "D2", "C9", "C6", "C2"]
north_cards = [Card.from_str(card_str) for card_str in north_card_strings]
north_hand = PlayerHand.from_cards(north_cards)

print(f"{club_nine=}, {diamond_three=}")
print(f"{south_hand=}")
print(f"(club_nine in north_hand.cards): {club_nine in north_hand.cards}")
```
```console
club_nine=C9, diamond_three=D3
south_hand=PlayerHand(SK S3 S2 | HK H7 H2 | DA DT D4 | CK C8 C7 C4)
(club_nine in north_hand.cards): True
```
#### What's Your Deal?
```python
east_hand = PlayerHand.from_string_lists(list("JT5"), list("K9865"), list("J8"), list("Q97"))
west_hand = PlayerHand.from_string_lists(list("AQ3"), list("Q7"), list("A63"), list("JT864"))
deal = Deal(
    dealer=Direction.NORTH,
    ns_vulnerable=False,
    ew_vulnerable=False,
    hands={
        Direction.NORTH: north_hand,
        Direction.EAST: east_hand,
        Direction.SOUTH: south_hand,
        Direction.WEST: west_hand,
    },
)
print(f"{deal=}")
print(f"{deal.player_cards[Direction.NORTH]=}")
print(f"{deal.hands[Direction.SOUTH]=}")
```
```console
deal=Deal(
	dealer=Direction.NORTH, ns_vulnerable=False, ew_vulnerable=False
	North: PlayerHand(SA S5 | HQ HT H9 H5 H4 | DJ D3 D2 | C9 C6 C2)
	South: PlayerHand(SK S3 S2 | HK H7 H2 | DA DT D4 | CK C8 C7 C4)
	East: PlayerHand(SQ S9 S7 | HJ H8 | DK D9 D8 D6 D5 | CJ CT C5)
	West: PlayerHand(SJ ST S8 S6 S4 | HA H6 H3 | DQ D7 | CA CQ C3)
)
deal.player_cards[Direction.NORTH]=[SA, S5, HQ, HT, H9, H5, H4, DJ, D3, D2, C9, C6, C2]
deal.hands[Direction.SOUTH]=PlayerHand(SK S3 S2 | HK H7 H2 | DA DT D4 | CK C8 C7 C4)
```
### Next Week on Bridgebots
This concludes the introduction to Bridgebots, but we're really just getting started. In my next post I will show you how Bridgebots represents the bidding and play history, and how to use Bridgebots to consume and interact with real bridge data!