<script lang="ts">
    import { onMount } from 'svelte';

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-04/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        partOneResult = calculateTotalPoints(items);
        partTwoResult = calculateQuantityOfCards(items);
    };

    const calculateTotalPoints = (cards) => {
        const result = cards.map(card => calculateCardPoints(card));
        return result.reduce((a, b) => a + b, 0);
    }

    const calculateCardPoints = (card) => {
        const result = calculateWinningPlayedPoints(card);
        return result.winningPlayedNumbers.length == 0 ? 0 : Math.pow(2, result.winningPlayedNumbers.length - 1);
    }

    const calculateQuantityOfCards = (cards) => {
        const cardsMap = new Map<number, number>();

        for (let [index] of cards.entries()) {
            const cardNumber = index + 1;

            if (cardsMap.has(cardNumber)) {
                let currentCardQuantity = cardsMap.get(cardNumber);
                cardsMap.set(cardNumber, currentCardQuantity + 1);
            } else {
                cardsMap.set(cardNumber, 1);
            }

            const copies = calculateCopies(index, cards);
            for (const copyCardNumber of copies) {
                if (copyCardNumber > cards.length) {
                    continue;
                }

                const quantityToBeAdded = cardsMap.get(cardNumber);
                if (cardsMap.has(copyCardNumber)) {
                    let currentCopyQuantity = cardsMap.get(copyCardNumber);
                    cardsMap.set(copyCardNumber, currentCopyQuantity + quantityToBeAdded);
                } else {
                    cardsMap.set(copyCardNumber, quantityToBeAdded);
                }
            }
        }

        return Array.from(cardsMap.values()).reduce((a, b) => a + b, 0);
    }

    const calculateCopies = (cardIndex, cards) => {
        const card = cards[cardIndex];
        const result = calculateWinningPlayedPoints(card);
        const copies = [];

        for (let i = cardIndex + 1; i <= cardIndex + result.winningPlayedNumbers.length; i++) {
            copies.push(i + 1);
        }

        return copies;
    }

    const calculateWinningPlayedPoints = (card) => {
        const cardStart = card.indexOf(":");
        const groups = card.substring(cardStart + 1).split("|");

        const winningNumbers = groups[0].trim().split(" ").filter(n => n != "").map(n => Number(n.trim()));
        const playedNumbers = groups[1].trim().split(" ").filter(n => n != "").map(n => Number(n.trim()));

        const winningPlayedNumbers = playedNumbers.filter(n => winningNumbers.includes(n));

        return {
            winningNumbers: winningNumbers,
            playedNumbers: playedNumbers,
            winningPlayedNumbers: winningPlayedNumbers
        }
    }

</script>

First part: {partOneResult}
Second part: {partTwoResult}