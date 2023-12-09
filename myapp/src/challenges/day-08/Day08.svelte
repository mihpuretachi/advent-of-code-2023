<script lang="ts">
    import { onMount } from 'svelte';
    import { lcm } from "mathjs";

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-08/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        const navigator = new Navigator(items);
        // partOneResult = navigator.calculateStepsForHumans("AAA", "ZZZ");
        partTwoResult = navigator.calculateStepsForGhosts();
    };

    class Navigator {
        private map: Map<string, [string, string]>;
        private commands: string;

        constructor(source: string[]) {
            this.processSource(source);
        }

        processSource(source: string[]) {
            this.commands = source[0].trim();
            this.map = new Map<string, [string, string]>();

            for (let i = 2; i < source.length; i++) {
                const row = source[i];
                const parts = row.split("=");
                const key = parts[0].trim();
                const values = parts[1].trim().replace("(", "").replace(")", "").split(",");
                this.map.set(key, [values[0].trim(), values[1].trim()]);
            }
        }

        calculateStepsForHumans(initialValue: string, finalValueEndsInValue: string): number {
            const commands = this.commands.split("");
            let steps = 0;
            let commandIndex = 0;
            let currentValue = initialValue;
            while (!currentValue.endsWith(finalValueEndsInValue)) {
                currentValue = this.getNextStep(currentValue, commands[commandIndex]);
                steps++;
                commandIndex++;
                if (commandIndex >= this.commands.length) {
                    commandIndex = 0;
                }
            }

            return steps;
        }

        calculateStepsForGhosts(): number {
            const initialValues = Array.from(this.map.keys())?.filter(v => v.endsWith("A"));

            const stepsForEachValue = initialValues.map(v => this.calculateStepsForHumans(v, "Z"));

            return lcm(...stepsForEachValue);
        }

        getNextStep(value: string, command: string): string {
            const possibleDirections = this.map.get(value);
            return possibleDirections[command === "L" ? 0 : 1];
        }
    }


</script>

First part: {partOneResult}
Second part: {partTwoResult}