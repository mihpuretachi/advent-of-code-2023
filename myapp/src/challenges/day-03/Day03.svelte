<script lang="ts">
    import { onMount } from 'svelte';
    import { uniq as distinct } from 'underscore';

    const symbolButNotDotPattern = /[ `!@#$%^&*()_+\-=\[\]{};':"\\|,<>\/?~]/;

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    class PartNumber {
        x: number;
        y: number;
        value: number;

        get size() {
            return this.value.toString().length;
        }

        canBeFoundAt(x: number, y: number) {
            const xStart = this.x;
            const xEnd = this.x + this.size - 1;
            return y == this.y && x >= xStart && x <= xEnd;
        }
    }

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-03/source.txt');
        const text = await response.text();
        const rows = text.split('\r\n');
        const partNumbers = mapPartNumbers(rows);
        partOneResult = partNumbers.map(pn => pn.value).reduce((a, b) => a + b, 0);
        partTwoResult = calculateGearRatioSum(rows, partNumbers);
    };

    const isSymbolExcludingDot = (value) => {
        return symbolButNotDotPattern.test(value);
    }

    const mapPartNumbers = (rows) => {
        let partNumbers = [];

        for (const [rowsIndex, row] of rows.entries()) {
            const rowAbove = rows[rowsIndex - 1];
            const rowBelow = rows[rowsIndex + 1];

            let number: string = '';

            const chars = row.split("");

            for (const [charIndex, char] of chars.entries()) {
                const hasNextChar = chars[charIndex + 1] != null;
                if (char == '.' || isSymbolExcludingDot(char)) {
                    if (number != '') {
                        if (isPartNumber(row, rowAbove, rowBelow, charIndex, number)) {
                            const partNumber = new PartNumber();
                            partNumber.x = charIndex - number.length;
                            partNumber.y = rowsIndex;
                            partNumber.value = Number(number);
                            partNumbers.push(partNumber);
                        }
                        number = '';
                    }
                } else if (!hasNextChar && !isNaN(Number(char))) {
                    number += char;
                    if (isPartNumber(row, rowAbove, rowBelow, charIndex + 1, number)) {
                        const partNumber = new PartNumber();
                        partNumber.x = charIndex - number.length - 1;
                        partNumber.y = rowsIndex;
                        partNumber.value = Number(number);
                        partNumbers.push(partNumber);
                    }
                    number = '';
                } else {
                    number += char;
                }
            }
        }

        return partNumbers;
    }

    const calculateGearRatioSum = (rows, partNumbers) => {
        let sum = 0;

        for (const [rowsIndex, row] of rows.entries()) {
            const chars = row.split("");

            for (const [charIndex, char] of chars.entries()) {
                if (char == "*") {
                    // 1 2 3
                    // 4 * 5
                    // 6 7 8

                    const adjacentPartNumbers = distinct([
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex - 1, rowsIndex - 1)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex, rowsIndex - 1)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex + 1, rowsIndex - 1)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex - 1, rowsIndex)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex + 1, rowsIndex)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex - 1, rowsIndex + 1)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex, rowsIndex + 1)),
                        partNumbers.find(pn => pn.canBeFoundAt(charIndex + 1, rowsIndex + 1)),
                    ].filter(pn => pn != null));

                    if (adjacentPartNumbers.length >= 2) {
                        const gearRatio = adjacentPartNumbers.map(pn => pn.value).reduce((a, b) => a * b, 1);

                        sum += gearRatio;
                    }
                }
            }
        }
        return sum;
    }

    const isPartNumber = (row, rowAbove, rowBelow, charIndex, currentNumberString) => {
        const hasSymbolAbove = !rowAbove ? false : hasSymbol(rowAbove, charIndex - currentNumberString.length - 1, charIndex);
        const hasSymbolBelow = !rowBelow ? false : hasSymbol(rowBelow, charIndex - currentNumberString.length - 1, charIndex);
        const hasSymbolToTheLeft = charIndex == 0 ? false : isSymbolExcludingDot(row[charIndex - currentNumberString.length - 1]);
        const hasSymbolToTheRight = charIndex == row.length ? false : isSymbolExcludingDot(row[charIndex]);

        return hasSymbolAbove || hasSymbolBelow || hasSymbolToTheLeft || hasSymbolToTheRight;
    }

    const hasSymbol = (row, start, end) => {
        for (let index = start; index <= end; index++) {
            if (isSymbolExcludingDot(row[index])) {
                return true;
            }
        }
        return false;
    }
</script>

First part: {partOneResult}
Second part: {partTwoResult}