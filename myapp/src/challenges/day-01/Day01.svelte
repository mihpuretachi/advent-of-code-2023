<script lang="ts">
    import { onMount } from 'svelte';

    const numbersMap = new Map<string, number>([
        ["one", 1],
        ["two", 2],
        ["three", 3],
        ["four", 4],
        ["five", 5],
        ["six", 6],
        ["seven", 7],
        ["eight", 8],
        ["nine", 9],
    ]);

    let finalValue = 0;
    let specialFinalValue = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-01/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        finalValue = items.reduce((a, b) => a + findCalibrationValue(b), 0);
        specialFinalValue = items.reduce((a, b) => a + findSpecialCalibrationValue(b), 0);
    };

    const findCalibrationValue = (value) => {
        const values = value.split('').filter(i => !isNaN(Number(i)));
        return Number(`${values[0]}${values[values.length - 1]}`);
    }

    const findSpecialCalibrationValue = (value) => {
        // Check first from start to end
        for (let i = 1; i < value.length; i++) {
            const part = value.substring(0, i);
            if (hasNumber(part)) {
                break;
            }

            const {foundReplace, newValue} = findReplaceNeeded(value, part, false);
            if (foundReplace) {
                value = newValue;
                break;
            }
        }

        // Check first from end to start
        for (let i = value.length-1; i > 0; i--) {
            const part = value.substring(i, value.length);
            if (hasNumber(part)) {
                break;
            }

            const {foundReplace, newValue} = findReplaceNeeded(value, part, true);
            if (foundReplace) {
                value = newValue;
                break;
            }
        }

        return findCalibrationValue(value);
    }

    const findReplaceNeeded = (value, substring, mustReplaceLast) => {
        const specialNumbers = Array.from(numbersMap.keys());

        if (specialNumbers.some(n => substring.includes(n))) {
            const foundSpecialNumber = specialNumbers.find(n => substring.includes(n));
            const translatedValue = numbersMap.get(foundSpecialNumber);
            value = mustReplaceLast ? replaceLast(value, foundSpecialNumber, translatedValue) : value.replace(substring, translatedValue);
            return {foundReplace: true, newValue: value};
        }

        return {foundReplace: false};
    }

    const replaceLast = (str, pattern, replacement) => {
        const match =
            typeof pattern === 'string'
                ? pattern
                : (str.match(new RegExp(pattern.source, 'g')) || []).slice(-1)[0];
        if (!match) return str;
        const last = str.lastIndexOf(match);
        return last !== -1
            ? `${str.slice(0, last)}${replacement}${str.slice(last + match.length)}`
            : str;
    };

    const hasNumber = (value) => {
        return /\d/.test(value);
    }
</script>

First part: {finalValue}
Second part: {specialFinalValue}