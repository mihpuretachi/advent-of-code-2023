<script lang="ts">
    import { onMount } from 'svelte';
    import { first, last } from "underscore";
    import { sum } from "mathjs";

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-09/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        const oasis = new Oasis(items);
        partOneResult = oasis.calculateExtrapolatedValuesSum();
        partTwoResult = oasis.calculateBackwardsExtrapolatedValuesSum();
    };

    class Oasis {
        rows: number[][] = [];

        constructor(source: string[]) {
            this.processSource(source);
        }

        processSource(source: string[]) {
            this.rows = source.map(row => row.split(" ").map(v => Number(v)));
        }

        calculateExtrapolatedValuesSum(): number {
            return sum(this.rows.map(r => this.calculateExtrapolatedValue(r)));
        }

        calculateBackwardsExtrapolatedValuesSum(): number {
            return sum(this.rows.map(r => this.calculateBackwardsExtrapolatedValue(r)));
        }

        calculateExtrapolatedValue(row: number[]): number {
            const history = this.calculateExtrapolationHistory(row);
            const values = [0];
            for (let i = history.length - 1; i > 0; i--) {
                const currentRowLastValue = last(values);

                const previousRow = history[i - 1];
                const previousRowLastValue = last(previousRow);

                values.push(currentRowLastValue + previousRowLastValue);
            }
            return last(values) ?? 0;
        }

        calculateBackwardsExtrapolatedValue(row: number[]): number {
            const history = this.calculateExtrapolationHistory(row);
            const values = [0];
            for (let i = history.length - 1; i > 0; i--) {
                const currentRowLastValue = last(values);

                const previousRow = history[i - 1];
                const previousRowFirstValue = first(previousRow);

                values.push(previousRowFirstValue - currentRowLastValue);
            }
            return last(values) ?? 0;
        }

        calculateExtrapolationHistory(row: number[]): number[][] {
            const extrapolations: number[][] = [row];
            let currentExtrapolation = extrapolations[0];
            while (currentExtrapolation.some(n => n != 0)) {
                const nextExtrapolation = this.calculateNextExtrapolation(currentExtrapolation);
                extrapolations.push(nextExtrapolation);
                currentExtrapolation = nextExtrapolation;
            }
            return extrapolations;
        }

        calculateNextExtrapolation(row: number[]): number[] {
            const nextExtrapolation = [];
            row.reduce((a, b) => {
                nextExtrapolation.push(b - a);
                return b;
            });
            return nextExtrapolation;
        }
    }
</script>

First part: {partOneResult}
Second part: {partTwoResult}