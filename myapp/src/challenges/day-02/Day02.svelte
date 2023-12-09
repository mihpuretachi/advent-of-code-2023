<script lang="ts">
    import { onMount } from 'svelte';

    const possibleRed = 12;
    const possibleGreen = 13;
    const possibleBlue = 14;

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-02/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        partOneResult = calculatePossibleGamesSum(items);
        partTwoResult = items.reduce((a, b) => a + calculatePower(b), 0);
    };

    const calculatePossibleGamesSum = (games) => {
        return games.map(i => {
            const result = isPossibleGame(i);
            if (result.isPossible) {
                return result.gameId;
            }
            return 0;

        }).reduce((a, b) => a + b, 0);
    }

    const isPossibleGame = (game) => {
        const gameIdPattern = new RegExp("Game (\\d+): (\\s|\\d|\\w|,|;)*$");
        const gameId = Number(Array.from(gameIdPattern.exec(game))[1]);
        const gameStart = game.indexOf(":");
        const gameGroups = game.substring(gameStart + 1).split(";");

        for (const group of gameGroups) {
            const rounds = group.split(",");
            for (let round of rounds) {
                round = round.trim();
                const redCount = extractNumberFromRound("red", round);
                if (redCount > possibleRed) {
                    return {
                        gameId: gameId,
                        isPossible: false
                    };
                }
                const blueCount = extractNumberFromRound("blue", round);
                if (blueCount > possibleBlue) {
                    return {
                        gameId: gameId,
                        isPossible: false
                    };
                }
                const greenCount = extractNumberFromRound("green", round);
                if (greenCount > possibleGreen) {
                    return {
                        gameId: gameId,
                        isPossible: false
                    };
                }
            }
        }

        return {
            gameId: gameId,
            isPossible: true
        };
    }

    const calculatePower = (game) => {
        const gameStart = game.indexOf(":");
        const gameGroups = game.substring(gameStart + 1).split(";");

        let minRed = 0;
        let minBlue = 0;
        let minGreen = 0;

        for (const group of gameGroups) {
            const rounds = group.split(",");
            for (let round of rounds) {
                const currentRed = extractNumberFromRound("red", round);
                const currentGreen = extractNumberFromRound("green", round);
                const currentBlue = extractNumberFromRound("blue", round);

                minRed = Math.max(minRed, currentRed);
                minBlue = Math.max(minBlue, currentBlue);
                minGreen = Math.max(minGreen, currentGreen);
            }
        }

        return minRed * minBlue * minGreen;
    }

    const extractNumberFromRound = (colorName, value) => {
        const pattern = new RegExp(`(\\d+) (${colorName})$`);
        const result = pattern.exec(value);
        return !result ? 0 : Number(Array.from(result)[1]);
    }


</script>

First part: {partOneResult}
Second part: {partTwoResult}