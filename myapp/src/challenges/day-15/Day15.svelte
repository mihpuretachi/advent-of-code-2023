<script lang="ts">
    import { onMount } from 'svelte';
    import { sum } from "mathjs";

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-15/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        const sequence = new InitializationSequence(items[0]);
        partOneResult = sequence.sumTranslatedValues();
        partTwoResult = sequence.calculateFocusingPower();
    };

    class InitializationSequence {
        private steps: Step[] = [];
        private boxes: Box[] = [];

        constructor(source: string) {
            this.steps = source.split(",").map(s => new Step(s));
        }

        sumTranslatedValues(): number {
            return sum(this.steps.map(s => s.translatedValue));
        }

        calculateFocusingPower() {
            this.resetBoxes();
            this.steps.forEach(s => s.applyTransformation(this.boxes));
            return sum(this.boxes.map(b => b.calculateFocusingPower()));
        }

        private resetBoxes(): void {
            this.boxes = [];
            for (let i = 0; i < 256; i++) {
                this.boxes.push(new Box(i));
            }
        }
    }

    class Step {
        private readonly boxNumber: number;
        translatedValue = 0;


        constructor(private value: string) {
            this.translatedValue = this.translate(value);
            this.boxNumber = this.translate(Array.from(new RegExp("([a-z]+)").exec(value))[0]);
        }

        applyTransformation(boxes: Box[]) {
            const box = boxes[this.boxNumber];
            if (this.value.includes("=")) {
                const parts = this.value.split("=");
                const label = parts[0];
                const power = Number(parts[1]);
                box.addLens(label, power);
            } else {
                const parts = this.value.split("-");
                box.removeLens(parts[0]);
            }
        }

        private translate(value: string,): number {
            let currentValue = 0;
            for (const char of value.split("")) {
                currentValue = ((currentValue + char.charCodeAt(0)) * 17) % 256;
            }
            return currentValue;
        }
    }

    class Box {
        private lens: Lens[] = [];

        constructor(public number: number) {
        }

        addLens(label: string, power: number) {
            const existingLens = this.lens.find(l => l.label == label);
            if (existingLens) {
                existingLens.power = power;
            } else {
                this.lens.push(new Lens(label, power));
            }
        }

        removeLens(label: string) {
            const index = this.lens.findIndex(l => l.label == label);
            if (index >= 0) {
                this.lens.splice(index, 1);
            }
        }

        calculateFocusingPower(): number {
            let focusingPower = 0;

            for (const [index, lens] of this.lens.entries()) {
                focusingPower += (this.number + 1) * (index + 1) * lens.power;
            }

            return focusingPower;
        }
    }

    class Lens {
        constructor(public label, public power) {
        }
    }

</script>

First part: {partOneResult}
Second part:  {partTwoResult}
