<script>
    import { onDestroy, beforeUpdate} from 'svelte';
    import { fly, slide } from 'svelte/transition';

    let interval;
    let speed = 500;
    $: dirtySquares = 100
    let startTime;
    let cleaningDuration = 0;
    let showDone = false;
    let previousDurations = [];

    beforeUpdate(() => {
        room = room
        previousDurations = JSON.parse(localStorage.getItem("previousDurations"));
    })

    const createRoom = (maxX, maxY) => {
        let matrix = [];
        for (let y = 0; y < maxY; y++) {
            const b = [];
            for (let x = 0; x < maxX; x++) {

                const o = {
                    clean: false,
                    x,
                    y
                };
                b.push(o);
            }
            matrix.push(b)
        }
        return {
            matrix,
            dirtySquares: maxX * maxY
        };
    }
    let room = createRoom(10, 10);


    const getRandomNumber = max => {
        return Math.floor(Math.random() * max);
    }

    const getStartingPosition = () => {
        let randX = getRandomNumber(10);
        let randY = getRandomNumber(10);

        let xPos = 50 * randX + 5;
        let yPos = 50 * randY + 5;
        return {x: randX, y: randY, xPos, yPos}
    };


    let robotPos = getStartingPosition();


    const getValidMove = () => {
        let rand = getRandomNumber(4);
        if (rand === 0 && robotPos.y !== 0) {
            return rand;
        }
        else if (rand === 1 && robotPos.x !== 9) {
            return rand;
        }

        else if (rand === 2 && robotPos.y !== 9) {
            return rand;
        }

        else if (rand === 3 && robotPos.x !== 0) {
            return rand;
        }

        return getValidMove()
    }

    const moveRobot = () => {

        let validMove = getValidMove()

        switch(validMove) {
            case (0):
                robotPos.yPos -= 50
                robotPos.y -= 1
                break;
            case (1):
                robotPos.xPos += 50
                robotPos.x += 1
                break;
            case (2):
                robotPos.yPos += 50
                robotPos.y += 1
                break;
            case (3):
                robotPos.xPos -= 50
                robotPos.x -= 1
                break;
            default:
                robotPos.yPos -= 50
                robotPos.y -= 1
                break;
        }
    };

    const cleanSquare = () => {
        let square = room.matrix[robotPos.y][robotPos.x]
        if (!square.clean) {
            square.clean = true;
            room.dirtySquares -= 1;
        }
    }
    const stopCleaning = () => {
        clearInterval(interval)
    }

    const changeSpeed = () => {
        stopCleaning()
        startCleaning(undefined, true)
    }

    const addTimeToStore = () => {

        let storedDurations = JSON.parse(localStorage.getItem("previousDurations"));

        if (storedDurations && storedDurations.length) {
            let update = [...storedDurations, cleaningDuration];
            localStorage.setItem("previousDurations", JSON.stringify(update));
            return;
        }

        let create = [cleaningDuration];
        localStorage.setItem("previousDurations", JSON.stringify(create));
    }

    const squaresRemaining = (squares) => {
        dirtySquares = squares;
        if (squares === 0) {
            showDone = true;
            stopCleaning()
            addTimeToStore()
        }
    }

    const startCleaning = (e, speedChange = false) => {
        // doesnt support stopping the cleaning process.
        if (!speedChange) {
            startTime = Date.now();
        }
        showDone = false;

        interval = setInterval(() => {
            moveRobot()
            let square = room.matrix[robotPos.y][robotPos.x]
            if (!square.clean) {
                square.clean = true;
                room.dirtySquares -= 1;
            }
            cleaningDuration = ((Date.now() - startTime) / 1000).toFixed(0)
            squaresRemaining(room.dirtySquares)
        }, speed);
    }

    const reset = () => {
        stopCleaning()
        room = createRoom(10, 10);
        robotPos = getStartingPosition()
        dirtySquares = 100
        showDone = false;
    }

    onDestroy(() => {
        stopCleaning()
    })


</script>

<main>
    <div class="controls">
        <button on:click={startCleaning}>Start</button>
        <button on:click={stopCleaning}>Stop</button>
        <button on:click={reset}>Reset</button>
        <label>
            <h3>Speed: {speed}ms</h3>
            <input on:change={changeSpeed} bind:value={speed} type="range" min="10" max="1000" step="10">
        </label>
    </div>

    <h3>Squares left to clean: {dirtySquares}</h3>
    <h3>Time spent cleaning: {cleaningDuration} sec</h3>
    <div class="wrapper" >

        {#each room.matrix as boxes}
            {#each boxes as {clean, x, y}}
                <div class:clean class="box"></div>
            {/each}
        {/each}
        <span class="dot" style="left: {robotPos.xPos}px; top: {robotPos.yPos}px"></span>
    </div>

    {#if previousDurations}
        <div class="prev-duration-wrapper">
            <h3>Previous</h3>
            {#each previousDurations.slice(0, 10) as item}
                <div class="prev-duration-item" transition:slide>
                    {item} seconds
                </div>
            {/each}
        </div>
    {/if}

    {#if showDone}
        <div class="done-dialog" transition:fly="{{ y: 250, duration: 2000 }}">
            Cleaning Complete in {cleaningDuration} seconds!
            <br>
            <span>Time spent cleaning resets when stopped.</span>
        </div>
    {/if}
</main>

<style>

    .prev-duration-wrapper {
        position: absolute;
        width: 200px;
        top: 200px;
        margin: 20px;
    }

    .prev-duration-item {
        padding: 10px 0;
        border-top: 1px solid grey;
    }

    h3 {
        text-align: center;
    }

    .box {
        background-color: #d4a233;
        border: 1px solid #575757;
    }

    .box:nth-child(even) {

    }

    .clean {
        background-color: #96ff8b;
    }

    .wrapper {
        display: grid;
        width: 500px;
        height: 500px;
        position: absolute;
        top: 200px;
        left: calc(50% - 250px);
        grid-template-columns: repeat(10, auto);
    }

    .dot {
        height: 40px;
        width: 40px;
        background-color: #0d0d34;
        border-radius: 50%;
        position: absolute;
    }

    .controls {
        display: flex;
        justify-content: space-evenly;
    }

    .done-dialog {
        position: absolute;
        top: 750px;
        width: 100%;
        text-align: center;
        font-weight: 700;
        font-size: 32px;
    }

    span {
        font-weight: 400;
        font-size: 16px;
    }
</style>
