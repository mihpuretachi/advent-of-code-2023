<script lang="ts">
    import { onMount } from 'svelte';
    import { sum } from "mathjs";

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-13/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        // partOneResult = new Challenge(items, false).summarizeNotes();
        partTwoResult = new Challenge(items, true).summarizeNotes();
    };

    class Challenge {
        private maps: MirrorsMap[] = [];

        constructor(source: string[], mustConsiderSmudges: boolean) {
            let rows = [];
            for (const row of source) {
                if (row == "") {
                    this.maps.push(new MirrorsMap(rows, mustConsiderSmudges));
                    rows = [];
                    continue;
                }
                rows.push(row);
            }
            this.maps.push(new MirrorsMap(rows, mustConsiderSmudges)); // Last map does not have a final empty line
        }

        summarizeNotes(): number {
            return sum(this.maps.map(m => m.summarize()));
        }

    }

    class MirrorsMap {
        private rows: MirrorPoint[][] = [];
        private columns: MirrorPoint[][] = [];
        private reflectColumnsStartingFrom: number
        private reflectRowsStartingFrom: number
        private erasedSmudge = false;


        constructor(private source: string[], private mustConsiderSmudges: boolean) {
            this.createRowsAndColumns(source);
            this.scanHorizontalReflections();
            if (this.reflectRowsStartingFrom == undefined) {
                this.scanVerticalReflections();
            }
        }

        private createRowsAndColumns(source: string[]) {
            for (const [y, row] of source.entries()) {
                const pointsRow = [];
                for (const [x, char] of row.split("").entries()) {
                    pointsRow.push(new MirrorPoint(x, y, char));
                }
                this.rows.push(pointsRow);
            }

            const rowLength = source[0].length;

            for (let i = 0; i < rowLength; i++) {
                this.columns.push(this.rows.map(r => r.find(c => c.x == i)));
            }
        }

        private scanVerticalReflections(): void {
            for (const [index, column] of this.columns.entries()) {
                const nextColumn = this.columns[index + 1];
                if (!nextColumn) {
                    break;
                }
                const differences = this.countDifferences(column, nextColumn);
                if (differences == 0 || (differences == 1 && !this.erasedSmudge)) {
                    if (this.verticalReflectionMirrorsPerfectlyStartingFrom(index)) {
                        this.reflectColumnsStartingFrom = index;
                        break;
                    }
                }
            }
        }

        private verticalReflectionMirrorsPerfectlyStartingFrom(x: number) {
            let reflects = true;
            let count = 0;
            let currentX = x;
            while (currentX >= 0) {
                const currentColumn = this.columns[currentX];
                const possibleReflection = this.columns[currentX + 1 + (count * 2)];
                if (!possibleReflection) {
                    break; // Pattern ended
                }
                const differences = this.countDifferences(currentColumn, possibleReflection);
                if (differences == 0) {
                    count += 1;
                    currentX -= 1;
                } else if (this.mustConsiderSmudges && !this.erasedSmudge && differences == 1) {
                    this.clearSmudgeBetween(currentColumn, possibleReflection);
                    count += 1;
                    currentX -= 1;
                } else {
                    reflects = false;
                    this.erasedSmudge = false; // Clear smudge didn't work
                    break;
                }
            }
            return reflects;
        }

        private clearSmudgeBetween(rowOrColumn: MirrorPoint[], anotherRowOrColumn: MirrorPoint[]) {
            for (const [index, point] of rowOrColumn.entries()) {
                const anotherPoint = anotherRowOrColumn[index];
                if (point.symbol != anotherPoint.symbol) {
                    point.symbol = anotherPoint.symbol;
                    break;
                }
            }
            this.erasedSmudge = true;
        }

        private scanHorizontalReflections(): void {
            for (const [index, row] of this.rows.entries()) {
                const nextRow = this.rows[index + 1];
                if (!nextRow) {
                    break;
                }
                const differences = this.countDifferences(row, nextRow);
                if (differences == 0 || (differences == 1 && !this.erasedSmudge)) {
                    if (this.horizontalReflectionMirrorsPerfectly(index)) {
                        this.reflectRowsStartingFrom = index;
                        break;
                    }
                }
            }
        }

        private horizontalReflectionMirrorsPerfectly(y: number): boolean {
            let reflects = true;
            let count = 0;
            let currentY = y;
            while (currentY >= 0) {
                const currentRow = this.rows[currentY];
                const possibleReflection = this.rows[currentY + 1 + (count * 2)];
                if (!possibleReflection) {
                    break; // Pattern ended
                }
                const differences = this.countDifferences(currentRow, possibleReflection);
                if (differences == 0) {
                    count += 1;
                    currentY -= 1;
                } else if (this.mustConsiderSmudges && !this.erasedSmudge && differences == 1) {
                    this.clearSmudgeBetween(currentRow, possibleReflection);
                    count += 1;
                    currentY -= 1;

                } else {
                    reflects = false;
                    this.erasedSmudge = false; // Clear smudge didn't work
                    break;
                }
            }

            return reflects;
        }

        private countDifferences(rowOrColumn: MirrorPoint[], anotherRowOrColumn: MirrorPoint[]): number {
            let count = 0;

            for (const [index, point] of rowOrColumn.entries()) {
                const anotherPoint = anotherRowOrColumn[index];
                if (point.symbol != anotherPoint.symbol) {
                    count++;
                }
            }

            return count;
        }

        summarize() {
            return (this.reflectColumnsStartingFrom != undefined ? (this.reflectColumnsStartingFrom + 1) : 0)
                + (this.reflectRowsStartingFrom != undefined ? (100 * (this.reflectRowsStartingFrom + 1)) : 0);
        }
    }

    class MirrorPoint {
        constructor(public x: number, public y: number, public symbol: string) {
        }
    }
</script>

First part: {partOneResult}
Second part:  {partTwoResult}
