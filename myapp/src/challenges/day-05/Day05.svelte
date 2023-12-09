<script lang="ts">
    import { onMount } from 'svelte';

    let partOneResult = 0;
    let partTwoResult = 0;

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-05/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        partOneResult = calculateMinLocation(items, false);
        partTwoResult = calculateMinLocation(items, true);
    };

    const calculateMinLocation = (rows, mustUseSeedRange) => {
        let seeds = [];
        let currentStage: Stage;
        const stages: Stage[] = [];

        for (let [index, row] of rows.entries()) {
            if (index == 0) {
                seeds = processSeeds(row, mustUseSeedRange);
                continue;
            }

            if (row == "") {
                continue;
            }

            const containsNumber = new RegExp("\\d+").test(row);
            if (!containsNumber) {
                // Start stage
                const stage = new Stage();
                stages.push(stage);
                currentStage = stage;
                continue;
            }

            // Calculate stage mapping
            const mapping = processRange(row);
            currentStage?.conversions.push(mapping);
        }

        return findMinLocation(seeds, stages);
    }

    const processSeeds = (row, mustUseSeedRange) => {
        const seeds = [];
        const numbers = row.split(":")[1].trim().split(" ").map(v => Number(v));
        for (let i = 0; i < numbers.length; i++) {
            if (mustUseSeedRange) {
                seeds.push(new LocationRange(numbers[i], numbers[i + 1]));
                i++; // jump to next pair
            } else {
                seeds.push(new LocationRange(numbers[i], 1));
            }
        }
        return seeds;
    }

    const processRange = (row) => {
        const values = row.split(" ");
        const destinationRangeStart = Number(values[0]);
        const sourceRangeStart = Number(values[1]);
        const rangeLength = Number(values[2]);
        return new ConversionRange(sourceRangeStart, destinationRangeStart, rangeLength);
    }

    const findMinLocation = (seeds: LocationRange[], stages: Stage[]) => {
        const minLocations: number[] = [];

        for (const seedRange of seeds) {
            let currentRanges = [seedRange];
            for (const stage of stages.values()) {
                currentRanges = stage.applyConversions(currentRanges);
            }
            minLocations.push(Math.min(...currentRanges.map(r => r.rangeStart)));
        }

        return Math.min(...minLocations);
    }

    class Stage {
        conversions: ConversionRange[] = [];

        applyConversions(ranges: LocationRange[]) {
            for (const conversion of this.conversions) {
                ranges = ranges.map(r => {
                    if (!r.processed) {
                        return conversion.apply(r);
                    } else {
                        return [r];
                    }
                }).reduce((a, b) => a.concat(b), []);
            }

            ranges.forEach(r => r.processed = false);

            return ranges;
        }
    }

    class ConversionRange {
        constructor(
            public sourceRangeStart: number,
            public destinationRangeStart: number,
            public rangeLength: number
        ) {
        }

        get sourceRangeEnd() {
            return this.sourceRangeStart + this.rangeLength - 1;
        }

        getIntersectionWith(range: LocationRange) {
            const start = Math.max(this.sourceRangeStart, range.rangeStart);
            const end = Math.min(this.sourceRangeEnd, range.rangeEnd);
            return new LocationRange(start, end - start + 1);
        }

        apply(locationRange: LocationRange): LocationRange[] {
            const destinations: LocationRange[] = [];

            // Handles range before stage
            if (locationRange.rangeStart < this.sourceRangeStart) {
                const rangeBeforeStageStart = Math.min(locationRange.rangeStart, this.sourceRangeStart);
                const rangeBeforeStageEnd = Math.min(this.sourceRangeStart, locationRange.rangeEnd);
                const rangeBeforeStageQuantity = rangeBeforeStageEnd - rangeBeforeStageStart + 1;
                const newLocationRange = new LocationRange(rangeBeforeStageStart, rangeBeforeStageQuantity);
                destinations.push(newLocationRange);
            }

            // Handles range after stage
            if (locationRange.rangeEnd > this.sourceRangeEnd) {
                const rangeAfterStageStart = Math.max(this.sourceRangeEnd, locationRange.rangeStart);
                const rangeAfterStageEnd = Math.max(this.sourceRangeEnd, locationRange.rangeEnd);
                const rangeAfterStageQuantity = rangeAfterStageEnd - rangeAfterStageStart + 1;
                const newLocationRange = new LocationRange(rangeAfterStageStart, rangeAfterStageQuantity);
                destinations.push(newLocationRange);
            }

            // Handles range inside stage
            const intersection = this.getIntersectionWith(locationRange);
            if (intersection.rangeLength > 0) {
                const start = this.destinationRangeStart + intersection.rangeStart - this.sourceRangeStart;
                destinations.push(new LocationRange(start, intersection.rangeLength, true));
            }

            return destinations;
        }
    }

    class LocationRange {
        processed = false;

        constructor(public rangeStart: number, public rangeLength: number, processed?: boolean | undefined) {
            if (processed) {
                this.processed = processed;
            }
        }

        get rangeEnd() {
            return this.rangeStart + this.rangeLength - 1;
        }
    }


</script>

First part: {partOneResult}
Second part: {partTwoResult}