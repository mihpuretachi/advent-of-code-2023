<script lang="ts">
    import { onMount } from 'svelte';
    import { abs, pow, sum } from "mathjs";

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-11/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        partOneResult = new Universe(items, 2, 1).calculateAllGalaxiesPairsShortestLengthSum();
        partTwoResult = new Universe(items, 10, 6).calculateAllGalaxiesPairsShortestLengthSum();
    };

    class Universe {
        galaxies: Galaxy[] = [];

        constructor(private source: string[], private expansionBase: number, private expansionPower: number) {
            this.expandUniverse();
            this.retrieveGalaxies();
        }

        private expandUniverse() {
            const map = this.source;
            for (let y = 0; y < map.length; y++) {
                const row = map[y];
                if (!row.includes("#")) {
                    this.replaceRowWithPoweredRow(y, row.length);
                }
            }

            let columnsNumber = map[0].length;

            for (let x = 0; x < columnsNumber; x++) {
                if (this.hasEmptyColumn(x)) {
                    this.replaceColumnWithPoweredColumn(x);
                }
            }
        }

        private hasEmptyColumn(index: number) {
            return this.source.every(row => row.split("")[index] !== "#");
        }

        private hasPoweredColumn(index: number) {
            return this.source.every(row => row.split("")[index] === this.expansionPower.toString());
        }

        private hasPoweredRow(index: number) {
            return this.source[index].split("").every(s => s === this.expansionPower.toString());
        }

        private replaceRowWithPoweredRow(index: number, lineLength: number) {
            const newLine = this.expansionPower.toString().repeat(lineLength);
            this.source.splice(index, 1, newLine);
        }

        private replaceColumnWithPoweredColumn(index: number) {
            for (const [i, row] of this.source.entries()) {
                const newRow = row.slice(0, index) + this.expansionPower + row.slice(index + 1);
                this.source.splice(i, 1, newRow);
            }
        }

        private retrieveGalaxies(): void {
            const map = this.source;
            for (let y = 0; y < map.length; y++) {
                const row = map[y];
                const chars = row.split("");
                for (let x = 0; x < chars.length; x++) {
                    const char = chars[x];
                    if (char === "#") {
                        this.galaxies.push(new Galaxy(x, y));
                    }
                }
            }
        }

        calculateAllGalaxiesPairsShortestLengthSum(): number {
            let result = 0;
            for (const galaxy of this.galaxies) {
                const otherGalaxies = this.galaxies.filter(g => g.id != galaxy.id);
                result += sum(...otherGalaxies.map(o => this.calculateShortestPathBetween(galaxy, o)));
            }
            return result / 2;
        }

        private calculateShortestPathBetween(galaxy: Galaxy, anotherGalaxy: Galaxy): number {
            const poweredRows = this.findPoweredRowsBetween(Math.min(galaxy.y, anotherGalaxy.y) + 1, Math.max(galaxy.y, anotherGalaxy.y) - 1);
            const poweredColumns = this.findPoweredColumnsBetween(Math.min(galaxy.x, anotherGalaxy.x) + 1, Math.max(galaxy.x, anotherGalaxy.x) - 1);
            return abs(galaxy.x - anotherGalaxy.x)
                - poweredColumns
                + abs(galaxy.y - anotherGalaxy.y)
                - poweredRows
                + (poweredRows * pow(this.expansionBase, this.expansionPower))
                + (poweredColumns * pow(this.expansionBase, this.expansionPower));
        }

        private findPoweredRowsBetween(start: number, end: number): number {
            let poweredRows = 0;
            for (let y = start; y <= end; y++) {
                if (this.hasPoweredRow(y)) {
                    poweredRows++;
                }
            }
            return poweredRows;
        }

        private findPoweredColumnsBetween(start: number, end: number): number {
            let poweredColumns = 0;
            for (let x = start; x <= end; x++) {
                if (this.hasPoweredColumn(x)) {
                    poweredColumns++;
                }
            }
            return poweredColumns;
        }
    }

    class Galaxy {
        constructor(public x: number, public y: number) {
        }

        get id(): string {
            return `(${this.x}, ${this.y})`
        }
    }


</script>

First part: {partOneResult}
Second part:  {partTwoResult}
