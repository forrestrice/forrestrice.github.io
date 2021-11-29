---
layout: post
title:  "Introducing Bridgebots Part 2: Deal Records and PBNs"
date:   2021-09-18 12:45:09 -0700
categories: bridge bridgebots
---

Last week I [introduced](/posts/Introducting-Bridgebots-Part-1) a new bridge programming library: [Bridgebots](https://github.com/forrestrice/bridge-bots). I covered how to create and interact with Cards, Suits, and Deals. Today we'll look at how Bridgebots represents the auction, the play of the hand, the score, and other event metadata. Once we have all the building blocks for representing bridge matches, I'll show you how Bridgebots can consume one of the most popular bridge data formats: PBN files.

### Setting the Table
There are many pieces of data that could be tracked while playing bridge, but a few are non-negotiable. Those are:
1. Players
2. Auction
3. Declarer
4. Contract
5. Card play
6. Number of tricks taken

Depending on the use case for the data, other important fields might include:
1. Alerts and explanations of bids
2. The scoring format
3. The event name and date
4. Player convention cards
5. Commentary from observers

Bridgebots currently supports most of these fields using the `DealRecord`, `BoardRecord`, `Contract`, `Commentary`, `BidMetadata` classes.

`BidMetadata` captures alerts and explanations and has a reference to which bid the metadata applies to.
```python
from bridgebots import (
    BidMetadata,
    BiddingSuit,
    BoardRecord,
    Card,
    Commentary,
    Contract,
    DealRecord,
    Direction,
    from_pbn_deal,
)

bidding_metadata = BidMetadata(
    bid_index=2, 
    bid="2NT", 
    alerted=False, 
    explanation="Unusual No Trump: 2 5card minors"
)
```
`Commentary` is similar in that it can reference a specific bid or card played.
```python
comment = Commentary(bid_index=1, play_index=None, comment="Amazing!"),
```
`Contract` includes the suit, the level, and a count of the number of times the contract was doubled.
```python
contract = Contract(6, BiddingSuit.DIAMONDS, 1) # 6DX
```

All of these are inputs into `BoardRecord` - the record of the play of a deal at a single table.

```python
play_record_strings = [
        "H4", "H2", "HJ", "HA",
        "DA", "D4", "D3", "S3",
        "DJ", "D8", "D6", "S4",
        "DT", "D9", "D7", "S6",
        "D2", "C3", "DK", "SJ",
        "DQ", "C4", "D5", "S7",
        "S2", "H3", "SK", "SA",
        "HT", "HQ", "HK", "C2",
        "H5", "S5", "H9", "H8",
        "C5", "CT", "CA", "C6",
        "H7", "C7", "ST", "S8",
        "H6", "C9", "C8", "S9",
        "CQ", "CJ", "CK", "SQ",
        ]
board_record = BoardRecord(
    bidding_record=["PASS", "1H", "2NT", "PASS", "3NT", "PASS", "PASS", "PASS"],
    raw_bidding_record=["p", "1H", "2N", "p", "3N", "p", "p", "p"],
    play_record=[Card.from_str(c) for c in play_record_strings],
    declarer=Direction.NORTH,
    declarer_vulnerable=True,
    contract=Contract(3, BiddingSuit.NO_TRUMP, 0),
    tricks=6,
    scoring=None,
    names={
        Direction.NORTH: "smalark",
        Direction.SOUTH: "PrinceBen",
        Direction.EAST: "granola357",
        Direction.WEST: "Forrest_",
    },
    date="2020-03-13",
    event="QuickTricks Club Game",
    bidding_metadata=[
        BidMetadata(bid_index=2, bid="2NT", alerted=False, explanation="Unusual No Trump: 2 5card minors")
    ],
    commentary=None,
)
```
Finally a `DealRecord` is simply a combination of a `Deal` and a list of `BoardRecords` corresponding to all the times the deal was played.

```python
# Peeking ahead at how PBN data can be consumed
deal = deal_utils.from_pbn_deal("N", "None", "W:98.KT5.J75.KJT84 K753.Q94.AT84.62 QJ6.AJ8762.62.73 AT42.3.KQ93.AQ95")
deal_record = DealRecord(deal, [board_record])
print(deal_record)
```
```console
DealRecord(deal=Deal(
	dealer=Direction.NORTH, ns_vulnerable=False, ew_vulnerable=False
	North: PlayerHand(SK S7 S5 S3 | HQ H9 H4 | DA DT D8 D4 | C6 C2)
	South: PlayerHand(SA ST S4 S2 | H3 | DK DQ D9 D3 | CA CQ C9 C5)
	East: PlayerHand(SQ SJ S6 | HA HJ H8 H7 H6 H2 | D6 D2 | C7 C3)
	West: PlayerHand(S9 S8 | HK HT H5 | DJ D7 D5 | CK CJ CT C8 C4)
), board_records=[BoardRecord(bidding_record=['PASS', '1H', '2NT', 'PASS', '3NT', 'PASS', 'PASS', 'PASS'], raw_bidding_record=['p', '1H', '2N', 'p', '3N', 'p', 'p', 'p'], play_record=[H4, H2, HJ, HA, DA, D4, D3, S3, DJ, D8, D6, S4, DT, D9, D7, S6, D2, C3, DK, SJ, DQ, C4, D5, S7, S2, H3, SK, SA, HT, HQ, HK, C2, H5, S5, H9, H8, C5, CT, CA, C6, H7, C7, ST, S8, H6, C9, C8, S9, CQ, CJ, CK, SQ], declarer=NORTH, contract=Contract(level=3, suit=NO_TRUMP, doubled=0), tricks=6, scoring=None, names={NORTH: 'smalark', SOUTH: 'PrinceBen', EAST: 'granola357', WEST: 'Forrest_'}, date='2020-03-13', event='QuickTricks Club Game', bidding_metadata=[BidMetadata(bid_index=2, bid='2NT', alerted=False, explanation='Unusual No Trump: 2 5card minors')], commentary=None, score=-300)])
```

All of these classes are implemented as frozen [Python Dataclasses](https://docs.python.org/3/library/dataclasses.html) and should be treated as immuatable.

### Portable Bridge Notation
Portable Bridge Notation (PBN) is one of the most common bridge data formats. It contains all the information we discussed above that is needed for record-keeping, as well as some additional fields which Bridgebots does not yet support. It is designed to be human-readable, which is great for players, but this design decision means PBN is somewhat difficult for a bridge program to use. If you want to know all the details, you can read the full [PBN standard](https://www.tistis.nl/pbn/). Here's an example:

```text
[Event "Cavendish Pairs Day 2"]
[Site "Rio Hotel, Las Vegas, USA"]
[Date "2004.05.05"]
[Board "10"]
[West ""]
[North ""]
[East ""]
[South ""]
[Dealer "E"]
[Vulnerable "All"]
[Deal "W:63.K3.K9532.J963 T82.62.T764.KQ42 KQJ7.QJ754.AJ.AT A954.AT98.Q8.875"]
[Scoring "IMP;Cross"]
[Declarer "W"]
[Contract "3NT"]
[Result "9"]
[Auction "E"]
1H Pass 1S =1= Pass
2C ! Pass 2H =2= Pass
2S =3= Pass 3NT AP
[Note "1:0-4 !ss"]
[Note "2:less than 8 points"]
[Note "3:17+ with 4 !S"]
[Play "N"]
CQ CA C8 C3
H6 H4 HT HK
H2 HQ HA H3
CK CT C5 C6
D4 DJ DQ DK
C2 S7 C7 CJ
C4 H5 S4 C9
-  -  -  S6
*
[Stage "Round 4"]
[HomeTeam "Team 1"]
[VisitTeam "Team 2"]
[ScoreIMP "NS -241"]


```
Most of these fields are straightforward, but a few require some effort to consume programmatically. The `Auction` section places four bids on each text line, but alerts and explanations are interspersed. To understand what a `Note` node is referring to, the user needs to match the note number to the bid proceeding the explanation number from the auction. 

The `Play` section keeps each player's play in the same text column. This is great for scanning the play as a human, but a program needs to be able to understand which player won the previous trick in order to determine the order the cards must have been played in the current trick.

### Python Bridge Nuptials
PBNs was the data format I most wanted access to for my bridge data projects, so the original use case for Bridgebots was parsing PBNs. The `pbn` module provides this.

```python
from pathlib import Path
from typing import List
from bridgebots import DealRecord, parse_pbn

pbn_path = Path("path/to/above/example.pbn")
results: List[DealRecord] = parse_pbn(pbn_path)
print(results)
```
```console
[DealRecord(deal=Deal(
	dealer=Direction.EAST, ns_vulnerable=True, ew_vulnerable=True
	North: PlayerHand(ST S8 S2 | H6 H2 | DT D7 D6 D4 | CK CQ C4 C2)
	South: PlayerHand(SA S9 S5 S4 | HA HT H9 H8 | DQ D8 | C8 C7 C5)
	East: PlayerHand(SK SQ SJ S7 | HQ HJ H7 H5 H4 | DA DJ | CA CT)
	West: PlayerHand(S6 S3 | HK H3 | DK D9 D5 D3 D2 | CJ C9 C6 C3)
), board_records=[BoardRecord(bidding_record=['1H', 'PASS', '1S', 'PASS', '2C', 'PASS', '2H', 'PASS', '2S', 'PASS', '3NT', 'PASS', 'PASS', 'PASS'], raw_bidding_record=['1H', 'Pass', '1S', '=1=', 'Pass', '2C', '!', 'Pass', '2H', '=2=', 'Pass', '2S', '=3=', 'Pass', '3NT', 'AP'], play_record=[CQ, CA, C8, C3, H4, HT, HK, H6, H3, H2, HQ, HA, C5, C6, CK, CT, D4, DJ, DQ, DK, CJ, C2, S7, C7, C9, C4, H5, S4, S6], declarer=WEST, contract=Contract(level=3, suit=NO_TRUMP, doubled=0), tricks=9, scoring='IMP;Cross', names={NORTH: '', SOUTH: '', EAST: '', WEST: ''}, date='2004.05.05', event='Cavendish Pairs Day 2', bidding_metadata=[BidMetadata(bid_index=2, bid='1S', alerted=False, explanation='0-4 !ss'), BidMetadata(bid_index=4, bid='2C', alerted=True, explanation=None), BidMetadata(bid_index=6, bid='2H', alerted=False, explanation='less than 8 points'), BidMetadata(bid_index=8, bid='2S', alerted=False, explanation='17+ with 4 !S')], commentary=None, score=600)])]
```
### Bridgebots Activate!
Last week I promised that I would show you Bridgebots in action with real bridge data. There is an amazing repository of PBN records at [Bridge Toernooi](http://bridgetoernooi.com) (Dutch for Tournament) covering many years of the Bermuda Bowl, Cavendish, Spingold, Vanderbilt and National matches. Let's start with the 2017 Bermuda Bowl which can be [downloaded](http://bridgetoernooi.com/index.php/home/download) as a single zipped PBN file.

First we load and process the pbn file.
```python
from collections import defaultdict
from pathlib import Path
from bridgebots import DealRecord, parse_pbn

pbn_path = Path("/path/to/bermuda_bowl_2017.pbn")
results = parse_pbn(pbn_path)
```
Now we can use Bridgebots to write simple programs to find data we're interested in. In 2017 USA2 consisting of Martin Fleisher, Joe Grue, Chip Martel, Brad Moss, Jacek Pszczoła, Michael Rosenberg, and Jan Martel (npc) won the main event. Michael Rosenberg is famous for his declarer play, so let's collect all the boards where he declared.

```python
rosenberg_deals = []
for deal_record in results:
    for board_record in deal_record.board_records:
        if board_record.names[board_record.declarer] == "Mic Rosenb":
            rosenberg_deals.append(DealRecord(deal_record.deal, [board_record]))
print(len(rosenberg_deals))
```
```console
47
```
Now let's see how often he made his contract.
```python
print(sum(deal_record.board_records[0].score >= 0 for deal_record in rosenberg_deals))
```
```console
31
```
Not bad!

Say I'm interested in going over every slam in the tournament.
```python
slam_deal_dict = defaultdict(list)
for deal_record in results:
    for board_record in deal_record.board_records:
        if board_record.contract.level >= 6:
            slam_deal_dict[deal_record.deal].append(board_record)
slam_deal_records = [DealRecord(deal, board_records) for deal, board_records in slam_deal_dict.items()]
print("deals with slams:", len(slam_deal_records))
print("boards with slams:", sum(len(dr.board_records) for dr in slam_deal_records))
```
```console
deals with slams: 42
boards with slams: 113
```
[Slammin!](https://youtu.be/mMqMCMcwO8o?t=15) 

That's all for today. Stay tuned for Part III where we will dive into the wonders of more bridge data formats: LIN (used by BBO) and JSON. 
    

