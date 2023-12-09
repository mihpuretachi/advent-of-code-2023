<script lang="ts">
    import { onMount } from 'svelte';

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-07/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        // partOneResult = new CamelCardsGame(items, false).calculateTotalWinning();
        partTwoResult = new CamelCardsGame(items, true).calculateTotalWinning();
    };

    class CamelCardsGame {
        private plays: CamelCardsPlay[] = [];

        constructor(source: string[], useJokers: boolean) {
            this.processSource(source, useJokers);
        }

        processSource(source: string[], useJokers: boolean) {
            this.plays = source.map(s => {
                const parts = s.split(" ");
                return new CamelCardsPlay(parts[0], Number(parts[1]), useJokers);
            });
        }

        calculateTotalWinning(): number {
            const ranking = this.plays.sort((p1, p2) => p1.compareWith(p2));
            let totalWinning = 0;

            for (const [index, value] of ranking.entries()) {
                totalWinning += (ranking.length - index) * value.bid;
            }

            return totalWinning;
        }
    }

    class CamelCardsPlay {
        private readonly cardPowers = new Map<string, [number, number]>([
            ["A", [13, 13]],
            ["K", [12, 12]],
            ["Q", [11, 11]],
            ["J", [10, 1]],
            ["T", [9, 10]],
            ["9", [8, 9]],
            ["8", [7, 8]],
            ["7", [6, 7]],
            ["6", [5, 6]],
            ["5", [4, 5]],
            ["4", [3, 4]],
            ["3", [2, 3]],
            ["2", [1, 2]]
        ]);

        private readonly playPowers = new Map<PlayType, number>([
            [PlayType.FiveOfAKind, 7],
            [PlayType.FourOfAKind, 6],
            [PlayType.FullHouse, 5],
            [PlayType.ThreeOfAKind, 4],
            [PlayType.TwoPair, 3],
            [PlayType.OnePair, 2],
            [PlayType.HighCard, 1]
        ]);

        private handMap = new Map<string, number>();

        constructor(public hand: string, public bid: number, private useJokers: boolean) {
            hand.split("").forEach(l => {
                if (this.handMap.has(l)) {
                    this.handMap.set(l, this.handMap.get(l) + 1);
                } else {
                    this.handMap.set(l, 1);
                }
            });
        }

        get playPower(): number {
            return this.playPowers.get(this.playType);
        }

        get playType(): PlayType {
            const keys = Array.from(this.handMap.keys());
            const jCards = this.handMap.has("J") ? this.handMap.get("J") : 0;

            if (keys.length == 1) {
                return PlayType.FiveOfAKind;
            }

            if (keys.length == 5) {
                return this.useJokers && jCards > 0 ? PlayType.OnePair : PlayType.HighCard;
            }

            if (keys.length == 4) {
                return this.useJokers ? (jCards == 0 ? PlayType.OnePair : PlayType.ThreeOfAKind) : PlayType.OnePair
            }

            const values = Array.from(this.handMap.values());

            if (keys.length == 3) {
                if (values.some(v => v === 3)) {
                    return this.useJokers ? (jCards === 0 ? PlayType.ThreeOfAKind : PlayType.FourOfAKind) : PlayType.ThreeOfAKind;
                }
                return this.useJokers ? (jCards == 0 ? PlayType.TwoPair : (jCards == 1 ? PlayType.FullHouse : PlayType.FourOfAKind)) : PlayType.TwoPair;
            }

            if (values.some(v => v === 4)) {
                return this.useJokers ? (jCards == 4 ? PlayType.FiveOfAKind : (jCards == 1 ? PlayType.FiveOfAKind : PlayType.FourOfAKind)) : PlayType.FourOfAKind;
            }

            return this.useJokers ? (jCards == 2 ? PlayType.FiveOfAKind : (jCards == 3 ? PlayType.FiveOfAKind : PlayType.FullHouse)) : PlayType.FullHouse;
        }

        compareWith(anotherPlay: CamelCardsPlay): number {
            const playPower = this.playPower;
            const anotherPlayPower = anotherPlay.playPower;

            if (playPower === anotherPlayPower) {
                return this.compareHandPower(anotherPlay.hand);
            }

            return playPower > anotherPlayPower ? -1 : 1;
        }

        compareHandPower(anotherHand: string): number {
            const cards = this.hand.split("");
            const anotherCards = anotherHand.split("");
            for (let i = 0; i < this.hand.length; i++) {
                const card = cards[i];
                const anotherCard = anotherCards[i];
                if (card == anotherCard) {
                    continue;
                }

                const cardPowers = this.cardPowers.get(card);
                const anotherCardPower = this.cardPowers.get(anotherCard);
                return cardPowers[this.useJokers ? 1 : 0] > anotherCardPower[this.useJokers ? 1 : 0] ? -1 : 1;
            }
        }
    }

    enum PlayType {
        FiveOfAKind,
        FourOfAKind,
        FullHouse,
        ThreeOfAKind,
        TwoPair,
        OnePair,
        HighCard
    }
</script>

First part: {partOneResult}
Second part: {partTwoResult}