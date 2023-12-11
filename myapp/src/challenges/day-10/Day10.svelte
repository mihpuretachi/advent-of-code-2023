<script lang="ts">
    import { onMount } from 'svelte';

    let partOneResult = 0;
    let partTwoResult = "";

    onMount(() => {
        loadResult();
    });

    const loadResult = async () => {
        const response = await fetch('src/challenges/day-10/source.txt');
        const text = await response.text();
        const items = text.split('\r\n');
        const pipeMap = new PipeMap(items);
        pipeMap.buildLoop();
        partOneResult = pipeMap.loop.stepsToMostDistantPipe;
        partTwoResult = pipeMap.drawLoop();
    };

    class PipeMap {
        points: Point[][] = [];
        initialPipe: Pipe;
        loop: PipePath;

        constructor(source: string[]) {
            this.processSource(source);
        }

        private processSource(source: string[]) {
            for (let y = 0; y < source.length; y++) {
                const pipeSymbols = source[y].split("");
                const currentPoints = [];
                this.points.push(currentPoints);
                for (let x = 0; x < pipeSymbols.length; x++) {
                    const pipeSymbol = pipeSymbols[x];
                    if (pipeSymbol == ".") {
                        currentPoints.push(new Ground(x, y));
                    } else {
                        const pipe = new Pipe(pipeSymbol, x, y);
                        this.processConnectionsFor(pipe);
                        if (pipe.isStartingPoint) {
                            this.initialPipe = pipe;
                        }
                        currentPoints.push(pipe);
                    }
                }
            }
        }

        buildLoop(): void {
            const loop = new PipePath();
            loop.addPipe(this.initialPipe);

            let completedLoop = false;
            let currentPipe = this.initialPipe;

            while (!completedLoop) {
                const nextPipe = currentPipe.connectedPipes.find(p => !loop.containsPipe(p));

                if (!nextPipe) {
                    completedLoop = true;
                } else {
                    loop.addPipe(nextPipe);
                    currentPipe = nextPipe;
                }
            }

            this.loop = loop;
        }

        private processConnectionsFor(pipe: Pipe): void {
            const pipeAbove = this.getPipeAbove(pipe);
            const pipeToTheLeft = this.getPipeToTheLeft(pipe);

            switch (pipe.symbol) {
                case "|" :
                    if (pipeAbove && pipeAbove.canConnectAboveOf(pipe)) {
                        pipe.connectTo(pipeAbove);
                        pipeAbove.connectTo(pipe);
                    }
                    break;
                case "-":
                    if (pipeToTheLeft && pipeToTheLeft.canConnectFromTheLeft(pipe)) {
                        pipe.connectTo(pipeToTheLeft);
                        pipeToTheLeft.connectTo(pipe);
                    }
                    break;
                case "L":
                    if (pipeAbove && pipeAbove.canConnectAboveOf(pipe)) {
                        pipe.connectTo(pipeAbove);
                        pipeAbove.connectTo(pipe);
                    }
                    break;
                case "J":
                    if (pipeAbove && pipeAbove.canConnectAboveOf(pipe)) {
                        pipe.connectTo(pipeAbove);
                        pipeAbove.connectTo(pipe);
                    }
                    if (pipeToTheLeft && pipeToTheLeft.canConnectFromTheLeft(pipe)) {
                        pipe.connectTo(pipeToTheLeft);
                        pipeToTheLeft.connectTo(pipe);
                    }
                    break;
                case "7":
                    if (pipeToTheLeft && pipeToTheLeft.canConnectFromTheLeft(pipe)) {
                        pipe.connectTo(pipeToTheLeft);
                        pipeToTheLeft.connectTo(pipe);
                    }
                    break;
                case "F":
                    break;
                case "S":
                    pipe.isStartingPoint = true;
                    pipe.stepsFromStart = 0;
                    if (pipeAbove && pipeAbove.canConnectAboveOf(pipe)) {
                        pipe.connectTo(pipeAbove);
                        pipeAbove.connectTo(pipe);
                    }
                    if (pipeToTheLeft && pipeToTheLeft.canConnectFromTheLeft(pipe)) {
                        pipe.connectTo(pipeToTheLeft);
                        pipeToTheLeft.connectTo(pipe);
                    }
                    break;
                default:
                    throw new Error(`Invalid pipe: ${pipe.symbol}`)
            }
        }

        // Points
        private getPointAbove(point: Point): Point | undefined {
            const rowAbove = this.points[point.y - 1];
            if (rowAbove) {
                return rowAbove[point.x];
            }
            return undefined;
        }

        private getPointToTheLeft(point: Point): Point | undefined {
            return this.points[point.y][point.x - 1];
        }

        private getPointBelow(point: Point): Point | undefined {
            const rowBelow = this.points[point.y + 1];
            if (rowBelow) {
                return rowBelow[point.x];
            }
            return undefined;
        }

        private getPointToTheRight(point: Point): Point | undefined {
            return this.points[point.y][point.x + 1];
        }

        // Pipes
        private getPipeAbove(pipe: Pipe): Pipe | undefined {
            const point = this.getPointAbove(pipe);
            if (point instanceof Pipe) {
                return point as Pipe
            }
            return undefined;
        }

        private getPipeToTheLeft(pipe: Pipe): Pipe | undefined {
            const point = this.getPointToTheLeft(pipe);
            if (point instanceof Pipe) {
                return point as Pipe;
            }
            return undefined;
        }

        drawLoop(): string {
            let drawing = "";
            for (const [index, row] of this.points.entries()) {
                drawing += ('000' + (index + 1)).substr(-3)
                for (const point of row) {
                    drawing += (this.loop.containsPipeIn(point) ? (point as Pipe).symbol : "*")
                        .replaceAll("L", "└")
                        .replaceAll("7", "┐")
                        .replaceAll("F", "┌")
                        .replaceAll("J", "┘")
                        .replaceAll("-", "─");
                }
                drawing += "\r\n";
            }
            return drawing;
        }
    }

    class PipePath {
        private readonly pipes: Pipe[] = [];

        containsPipe(pipe: Pipe): boolean {
            return pipe && this.pipes.some(p => p.x == pipe.x && p.y == pipe.y);
        }

        containsPipeIn(point: Point): boolean {
            return this.pipes.some(p => p.x == point.x && p.y == point.y);
        }

        addPipe(pipe: Pipe): void {
            this.pipes.push(pipe);
        }

        get stepsToMostDistantPipe() {
            return this.pipes.length / 2;
        }
    }

    class Pipe implements Point {
        connectedPipes: Pipe[] = [];
        stepsFromStart: number;
        isStartingPoint: boolean;

        constructor(public symbol: string, public x: number, public y: number) {
        }

        connectTo(pipe: Pipe): void {
            this.connectedPipes.push(pipe);
        }

        canConnectAboveOf(pipe: Pipe): boolean {
            switch (this.symbol) {
                case "|" :
                case "7":
                case "F":
                    return ["|", "L", "J"].includes(pipe.symbol);
                case "-":
                case "L":
                case "J":
                    return false;
                case "S":
                    return true;
            }
        }

        canConnectFromTheLeft(pipe: Pipe): boolean {
            switch (this.symbol) {
                case "-" :
                case "L":
                case "F":
                    return ["-", "7", "J"].includes(pipe.symbol);
                case "|":
                case "7":
                case "J":
                    return false;
                case "S":
                    return true;
            }
        }
    }

    class Ground implements Point {
        constructor(public x: number, public y: number) {
        }
    }

    interface Point {
        x: number;
        y: number;
    }
</script>

First part: {partOneResult}
<br/>
Second part:
<br/>
<pre style="font-family: monospace; word-wrap: break-word; white-space: pre-wrap;">
{partTwoResult}
</pre>