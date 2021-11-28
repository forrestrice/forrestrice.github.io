---
layout: post
title:  "Introducing Bridgebots Part 3: LIN files and JSON"
date:   2021-11-27 12:45:09 -0700
categories: bridge bridgebots
---
In my [last post](/posts/Introducing-Bridgebots-Part-2) I covered how [Bridgebots](https://github.com/forrestrice/bridge-bots) records the auction and the play of the hand, and how it can be used to import and analyze results from PBN files. Today we will expand that functionality to LIN and JSON files, and your long-winded introduction to Bridgebots will be complete! 

### LINsanity
[Bridge Base Online](https://www.bridgebase.com)(BBO) is a popular site for playing bridge both competitively and casually. One of its greatest features is the history it saves for each user and tournament. Here are some helpful places to find these:
- [Player Hand Records ](https://www.bridgebase.com/myhands/index.php?&from_login=0) (requires account)
- [Vugraph Archives](https://www.bridgebase.com/vugraph_archives/vugraph_archives.php)
- [Recent Tournaments](https://www.bridgebase.com/tourneyhistory/)

BBO provides an interactive `Movie` viewing option or a `Lin` file for download. I don't actually know what LIN stands for (if you do let me know!). It contains all the juicy details we need to record the play of a bridge hand, but it takes a little bit of puzzling to figure out what each part means. Here's an example:

```text
pn|SBlumlein,Forrest_,grychn48,granola357|st||md|2S9HJ872DQT64CQJ43,SAK2HA95DAJ875C92,SQ765HKQ64D32CAK6,|rh||ah|Board 12|sv|n|mb|1N|an|15-17|mb|p|mb|2H|an|spades |mb|p|mb|2S|mb|p|mb|p|mb|p|pg||pc|CA|pc|C5|pc|C4|pc|C2|pg||pc|CK|pc|C7|pc|C3|pc|C9|pg||pc|C6|pc|CT|pc|CJ|pc|S2|pg||pc|SA|pc|S5|pc|S3|pc|S9|pg||pc|SK|pc|S6|pc|S4|pc|D4|pg||pc|D5|pc|D3|pc|DK|pc|D6|pg||pc|D9|pc|DQ|pc|DA|pc|D2|pg||pc|DJ|pc|S7|pc|S8|pc|DT|pg||pc|H3|pc|H2|pc|HA|pc|H4|pg||pc|D8|pc|SQ|pc|HT|pc|H7|pg||pc|HK|pc|ST|pc|H8|pc|H5|pg||pc|SJ|pc|HJ|pc|H9|pc|H6|pg||pc|C8|pc|CQ|pc|D7|pc|HQ|pg||
```
[BBO movie for reference](https://www.bridgebase.com/tools/handviewer.html?bbo=y&lin=pn|SBlumlein,Forrest_,grychn48,granola357|st%7C%7Cmd%7C2S9HJ872DQT64CQJ43%2CSAK2HA95DAJ875C92%2CSQ765HKQ64D32CAK6%2C%7Crh%7C%7Cah%7CBoard%2012%7Csv%7Cn%7Cmb%7C1N%7Can%7C15-17%7Cmb%7Cp%7Cmb%7C2H%7Can%7Cspades%20%7Cmb%7Cp%7Cmb%7C2S%7Cmb%7Cp%7Cmb%7Cp%7Cmb%7Cp%7Cpc%7CCA%7Cpc%7CC5%7Cpc%7CC4%7Cpc%7CC2%7Cpc%7CCK%7Cpc%7CC7%7Cpc%7CC3%7Cpc%7CC9%7Cpc%7CC6%7Cpc%7CCT%7Cpc%7CCJ%7Cpc%7CS2%7Cpc%7CSA%7Cpc%7CS5%7Cpc%7CS3%7Cpc%7CS9%7Cpc%7CSK%7Cpc%7CS6%7Cpc%7CS4%7Cpc%7CD4%7Cpc%7CD5%7Cpc%7CD3%7Cpc%7CDK%7Cpc%7CD6%7Cpc%7CD9%7Cpc%7CDQ%7Cpc%7CDA%7Cpc%7CD2%7Cpc%7CDJ%7Cpc%7CS7%7Cpc%7CS8%7Cpc%7CDT%7Cpc%7CH3%7Cpc%7CH2%7Cpc%7CHA%7Cpc%7CH4%7Cpc%7CD8%7Cpc%7CSQ%7Cpc%7CHT%7Cpc%7CH7%7Cpc%7CHK%7Cpc%7CST%7Cpc%7CH8%7Cpc%7CH5%7Cpc%7CSJ%7Cpc%7CHJ%7Cpc%7CH9%7Cpc%7CH6%7Cpc%7CC8%7Cpc%7CCQ%7Cpc%7CD7%7Cpc%7CHQ%7C)

Parsing it visually we can see the player names, the hands (well three of them), the board number, the vulnerability, the auction, and the play of the hand. LIN files suffer from some of the same issues that afflict PBNs: they are fairly human-readable but they are non-trivial to consume programmatically. Luckily Bridgebots can help us out!

```python
from pathlib import Path
from bridgebots import parse_multi_lin, parse_single_lin

lin_path = Path("path/to/above/example.lin")
results = parse_single_lin(lin_path)
print(results)
```

```console
[DealRecord(deal=Deal(
	dealer=Direction.WEST, ns_vulnerable=True, ew_vulnerable=False
	North: PlayerHand(SQ S7 S6 S5 | HK HQ H6 H4 | D3 D2 | CA CK C6)
	South: PlayerHand(S9 | HJ H8 H7 H2 | DQ DT D6 D4 | CQ CJ C4 C3)
	East: PlayerHand(SJ ST S8 S4 S3 | HT H3 | DK D9 | CT C8 C7 C5)
	West: PlayerHand(SA SK S2 | HA H9 H5 | DA DJ D8 D7 D5 | C9 C2)
), board_records=[BoardRecord(bidding_record=['1NT', 'PASS', '2H', 'PASS', '2S', 'PASS', 'PASS', 'PASS'], raw_bidding_record=['1N', 'p', '2H', 'p', '2S', 'p', 'p', 'p'], play_record=[CA, C5, C4, C2, CK, C7, C3, C9, C6, CT, CJ, S2, SA, S5, S3, S9, SK, S6, S4, D4, D5, D3, DK, D6, D9, DQ, DA, D2, DJ, S7, S8, DT, H3, H2, HA, H4, D8, SQ, HT, H7, HK, ST, H8, H5, SJ, HJ, H9, H6, C8, CQ, D7, HQ], declarer=WEST, contract=Contract(level=2, suit=SPADES, doubled=0), tricks=9, scoring=None, names={SOUTH: 'SBlumlein', WEST: 'Forrest_', NORTH: 'grychn48', EAST: 'granola357'}, date=None, event=None, bidding_metadata=[BidMetadata(bid_index=0, bid='1NT', alerted=False, explanation='15-17'), BidMetadata(bid_index=2, bid='2H', alerted=False, explanation='spades ')], commentary=None, score=140)])]
```
As we can see, Bridgebots successfully parsed the deal, bidding, play, and even included the alerts and explanations!

Bridgebots can also handle multi-board LINs produced by teams matches. For example [this segment](https://www.bridgebase.com/tools/handviewer.html?bbo=y&linurl=https://www.bridgebase.com/tools/vugraph_linfetch.php?id=14502) of the 2010 USBF semi-finals can be downloaded [here](https://www.bridgebase.com/tools/vugraph_linfetch.php?id=14502).

```python
lin_multi_path = Path("path/to/sample_multi.lin")
results = parse_multi_lin(lin_multi_path)
print(len(results))
```
```console
15
```
```python
print(results[0].board_records[0])
```
```console
BoardRecord(bidding_record=['1H', 'PASS', '3C', 'PASS', '4H', 'PASS', 'PASS', 'PASS'], raw_bidding_record=['1H', 'p', '3C!', 'p', '4H', 'p', 'p', 'p'], play_record=[C2, C3, CA, CJ, D7, D5, DA, D4, D6, DQ, D9, D3, C9, C4, CK, C5, H2, H9, HA, H5], declarer=EAST, contract=Contract(level=4, suit=HEARTS, doubled=0), tricks=10, scoring=None, names={SOUTH: 'Meckstroth', WEST: 'Levin', NORTH: 'Rodwell', EAST: 'Weinstein'}, date=None, event=None, bidding_metadata=[BidMetadata(bid_index=2, bid='3C', alerted=True, explanation='jacoby 2N')], commentary=[Commentary(bid_index=1, play_index=None, comment='caitlin: Hi all'), Commentary(bid_index=1, play_index=None, comment='vugraphzfk: Hi everyone and welcome back, this is the last segment for today'), Commentary(bid_index=1, play_index=None, comment='ahollan1: same boards played in both Semi-Final matches   use http://www.bbotv.com/vugraph/   to spy on all tables'), Commentary(bid_index=None, play_index=3, comment='ahollan1: ACBL Convention Cards for all Fleisher Team  http://usbf.org/index.php?option=com_content&task=view&id=659&Itemid=284'), Commentary(bid_index=None, play_index=3, comment='vdoubleu: Good ---1 quick hand down'), Commentary(bid_index=None, play_index=3, comment='ahollan1: ACBL CC and WBF CC from 2009 Bermuda Bowl for Nickell team at http://usbf.org/index.php?option=com_content&task=view&id=655&Itemid=284'), Commentary(bid_index=None, play_index=11, comment='vugraphzfk: you jinxed it, Val'), Commentary(bid_index=None, play_index=11, comment='vdoubleu: I noticed:-)')], score=420)
```
Bridgebots gives us all its usual goodness and also includes the expert commentary for your (and your computer's) viewing pleasure.

Finally, Bridgebots also includes the ability to create LIN strings and URLs from DealRecords. Let's load up our example Cavendish Pairs PBN from [Part 2](/posts/Introducing-Bridgebots-Part-2/) and take a look at it with the BBO interactive viewer!

```python
from pathlib import Path
from bridgebots import build_lin_str, build_lin_url, parse_pbn

pbn_path = Path("path/to/cavendish.pbn")
results = parse_pbn(pbn_path)
print(build_lin_str(results[0].deal, results[0].board_records[0]))
```

```console
pn|,,,|st||md|4SA954HAT98DQ8C875,S63HK3DK9532CJ963,ST82H62DT764CKQ42,SKQJ7HQJ754DAJCAT|sv|b|mb|1H|mb|p|mb|1S|an|0-4 !ss|mb|p|mb|2C!|mb|p|mb|2H|an|less than 8 points|mb|p|mb|2S|an|17+ with 4 !S|mb|p|mb|3N|mb|p|mb|p|mb|p|pg||pc|CQ|pc|CA|pc|C8|pc|C3|pg||pc|H4|pc|HT|pc|HK|pc|H6|pg||pc|H3|pc|H2|pc|HQ|pc|HA|pg||pc|C5|pc|C6|pc|CK|pc|CT|pg||pc|D4|pc|DJ|pc|DQ|pc|DK|pg||pc|CJ|pc|C2|pc|S7|pc|C7|pg||pc|C9|pc|C4|pc|H5|pc|S4|pg||pc|S6|mc|9|pg||
```
```python
print(build_lin_url(results[0].deal, results[0].board_records[0]))
```

```console
https://www.bridgebase.com/tools/handviewer.html?lin=pn%7C%2C%2C%2C%7Cst%7C%7Cmd%7C4SA954HAT98DQ8C875%2CS63HK3DK9532CJ963%2CST82H62DT764CKQ42%2CSKQJ7HQJ754DAJCAT%7Csv%7Cb%7Cmb%7C1H%7Cmb%7Cp%7Cmb%7C1S%7Can%7C0-4+%21ss%7Cmb%7Cp%7Cmb%7C2C%21%7Cmb%7Cp%7Cmb%7C2H%7Can%7Cless+than+8+points%7Cmb%7Cp%7Cmb%7C2S%7Can%7C17%2B+with+4+%21S%7Cmb%7Cp%7Cmb%7C3N%7Cmb%7Cp%7Cmb%7Cp%7Cmb%7Cp%7Cpg%7C%7Cpc%7CCQ%7Cpc%7CCA%7Cpc%7CC8%7Cpc%7CC3%7Cpg%7C%7Cpc%7CH4%7Cpc%7CHT%7Cpc%7CHK%7Cpc%7CH6%7Cpg%7C%7Cpc%7CH3%7Cpc%7CH2%7Cpc%7CHQ%7Cpc%7CHA%7Cpg%7C%7Cpc%7CC5%7Cpc%7CC6%7Cpc%7CCK%7Cpc%7CCT%7Cpg%7C%7Cpc%7CD4%7Cpc%7CDJ%7Cpc%7CDQ%7Cpc%7CDK%7Cpg%7C%7Cpc%7CCJ%7Cpc%7CC2%7Cpc%7CS7%7Cpc%7CC7%7Cpg%7C%7Cpc%7CC9%7Cpc%7CC4%7Cpc%7CH5%7Cpc%7CS4%7Cpg%7C%7Cpc%7CS6%7Cmc%7C9%7Cpg%7C%7C
```

Click [here](https://www.bridgebase.com/tools/handviewer.html?lin=pn%7C%2C%2C%2C%7Cst%7C%7Cmd%7C4SA954HAT98DQ8C875%2CS63HK3DK9532CJ963%2CST82H62DT764CKQ42%2CSKQJ7HQJ754DAJCAT%7Csv%7Cb%7Cmb%7C1H%7Cmb%7Cp%7Cmb%7C1S%7Can%7C0-4+%21ss%7Cmb%7Cp%7Cmb%7C2C%21%7Cmb%7Cp%7Cmb%7C2H%7Can%7Cless+than+8+points%7Cmb%7Cp%7Cmb%7C2S%7Can%7C17%2B+with+4+%21S%7Cmb%7Cp%7Cmb%7C3N%7Cmb%7Cp%7Cmb%7Cp%7Cmb%7Cp%7Cpg%7C%7Cpc%7CCQ%7Cpc%7CCA%7Cpc%7CC8%7Cpc%7CC3%7Cpg%7C%7Cpc%7CH4%7Cpc%7CHT%7Cpc%7CHK%7Cpc%7CH6%7Cpg%7C%7Cpc%7CH3%7Cpc%7CH2%7Cpc%7CHQ%7Cpc%7CHA%7Cpg%7C%7Cpc%7CC5%7Cpc%7CC6%7Cpc%7CCK%7Cpc%7CCT%7Cpg%7C%7Cpc%7CD4%7Cpc%7CDJ%7Cpc%7CDQ%7Cpc%7CDK%7Cpg%7C%7Cpc%7CCJ%7Cpc%7CC2%7Cpc%7CS7%7Cpc%7CC7%7Cpg%7C%7Cpc%7CC9%7Cpc%7CC4%7Cpc%7CH5%7Cpc%7CS4%7Cpg%7C%7Cpc%7CS6%7Cmc%7C9%7Cpg%7C%7C) to follow that link.

### JSON
As we have seen, both PBNs and LINs require significant effort for programs to parse. I wanted the ability to export Bridgebots data in a format that would be easily consumable by other programs. To that end, Bridgebots includes [Marshmallow](https://marshmallow.readthedocs.io/en/stable/) schemas for serializing/deserializing its data to/from JSON. These may not be perfect for other programmers bridge needs, but my hope is they will be useful for consuming translated PBN and LIN records. Check out this example:

```python
import json
from pathlib import Path
from bridgebots import DealRecordSchema, parse_single_lin

results = parse_single_lin(Path("path/to/sample.lin"))
deal_record_schema = DealRecordSchema(many=True)
deal_record_json = deal_record_schema.dump(results)
print(json.dumps(deal_record_json, indent=4))
```
```console
[
    {
        "deal": {
            "dealer": "W",
            "ns_vulnerable": true,
            "ew_vulnerable": false,
            "hands": {
                "S": [
                    "S9",
                    "HJ",
                    "H8",
                    "H7",
                    "H2",
                    "DQ",
                    "DT",
                    "D6",
                    "D4",
                    "CQ",
                    "CJ",
                    "C4",
                    "C3"
                ],
                "W": [
                    "SA",
                    "SK",
                    "S2",
                    "HA",
                    "H9",
                    "H5",
                    "DA",
                    "DJ",
                    "D8",
                    "D7",
                    "D5",
                    "C9",
                    "C2"
                ],
                "N": [
                    "SQ",
                    "S7",
                    "S6",
                    "S5",
                    "HK",
                    "HQ",
                    "H6",
                    "H4",
                    "D3",
                    "D2",
                    "CA",
                    "CK",
                    "C6"
                ],
                "E": [
                    "SJ",
                    "ST",
                    "S8",
                    "S4",
                    "S3",
                    "HT",
                    "H3",
                    "DK",
                    "D9",
                    "CT",
                    "C8",
                    "C7",
                    "C5"
                ]
            }
        },
        "board_records": [
            {
                "bidding_record": [
                    "1NT",
                    "PASS",
                    "2H",
                    "PASS",
                    "2S",
                    "PASS",
                    "PASS",
                    "PASS"
                ],
                "raw_bidding_record": [
                    "1N",
                    "p",
                    "2H",
                    "p",
                    "2S",
                    "p",
                    "p",
                    "p"
                ],
                "play_record": [
                    "CA",
                    "C5",
                    "C4",
                    "C2",
                    "CK",
                    "C7",
                    "C3",
                    "C9",
                    "C6",
                    "CT",
                    "CJ",
                    "S2",
                    "SA",
                    "S5",
                    "S3",
                    "S9",
                    "SK",
                    "S6",
                    "S4",
                    "D4",
                    "D5",
                    "D3",
                    "DK",
                    "D6",
                    "D9",
                    "DQ",
                    "DA",
                    "D2",
                    "DJ",
                    "S7",
                    "S8",
                    "DT",
                    "H3",
                    "H2",
                    "HA",
                    "H4",
                    "D8",
                    "SQ",
                    "HT",
                    "H7",
                    "HK",
                    "ST",
                    "H8",
                    "H5",
                    "SJ",
                    "HJ",
                    "H9",
                    "H6",
                    "C8",
                    "CQ",
                    "D7",
                    "HQ"
                ],
                "declarer": "W",
                "contract": "2S",
                "tricks": 9,
                "score": 140,
                "scoring": null,
                "names": {
                    "S": "SBlumlein",
                    "W": "Forrest_",
                    "N": "grychn48",
                    "E": "granola357"
                },
                "date": null,
                "event": null,
                "bidding_metadata": [
                    {
                        "bid_index": 0,
                        "bid": "1NT",
                        "alerted": false,
                        "explanation": "15-17"
                    },
                    {
                        "bid_index": 2,
                        "bid": "2H",
                        "alerted": false,
                        "explanation": "spades "
                    }
                ],
                "commentary": null
            }
        ]
    }
]
```

```python
reloaded_deal_record = deal_record_schema.load(deal_record_json)
print(f"{(results == reloaded_deal_record)=}")
```

```console
(results == reloaded_deal_record)=True
```

### Looking ahead
It has been great walking you through Bridgebots - thank you so much for reading if you made it this far. I put many hours of effort into its current capabilities, and I am proud of that work, but I hope that this is just the beginning. Here are some possible projects for the future:

- Ability to create PBNs and to read/write PBNv2 files
- Integration with [redeal](https://github.com/anntzer/redeal), another awesome open-source python bridge library. This would provide simulations and access to [DDS](https://github.com/dds-bridge/dds), a powerful open-source double-dummy solver.
- Data exploration with python tools like [pandas](https://github.com/pandas-dev/pandas) and [matplotlib](https://github.com/matplotlib/matplotlib). I did a little exploration of some bridge data [here](https://github.com/forrestrice/bridge-bots/blob/master/plot/bridge_analysis.ipynb).
- Creating ML models to analyze and predict bidding and play. I hope to make this the subject of my next post!
- A web-viewer for Bridgebots data

If you have ideas for potential applications of or improvements to Bridgebots, feel free to reach out or submit a pull request. 

Hope to see you at the bridge table!

-Forrest