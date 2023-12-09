<script lang="ts">
    import { onMount } from 'svelte';

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-06/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        partOneResult = multiplyWays(items);
        partTwoResult = calculateSingleRace(items);
    };

    const multiplyWays = (rows) => {
        const times = Array.from(rows[0].matchAll("\\d+")).map(match => Number(match[0]));
        const distances = Array.from(rows[1].matchAll("\\d+")).map(match => Number(match[0]));
        const ways = [];

        for (let i = 0; i < times.length; i++) {
            const time = times[i];
            const distance = distances[i];
            ways.push(calculateWaysToBeat(time, distance));
        }

        return ways.reduce((a, b) => a * b, 1);
    }

    const calculateSingleRace = (rows) => {
        const time = Number(Array.from(rows[0].matchAll("\\d+")).map(match => Number(match[0])).reduce((a, b) => a + b, ""));
        const distance = Number(Array.from(rows[1].matchAll("\\d+")).map(match => Number(match[0])).reduce((a, b) => a + b, ""));

        return calculateWaysToBeat(time, distance);
    }

    const calculateWaysToBeat = (time: number, distance: number) => {
        let waysToBeat = 0;
        for (let currentTime = 1; currentTime < time; currentTime++) {
            const distanceReached = calculateDistance(time, currentTime);
            if (distanceReached > distance) {
                waysToBeat++;
            }
        }
        return waysToBeat;
    }

    const calculateDistance = (timeToBeat: number, time: number) => {
        return (timeToBeat - time) * time;
    }

</script>

First part: {partOneResult}
Second part: {partTwoResult}