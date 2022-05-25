<script setup lang="ts">
    type Coordinate = {
        x: number;
        y: number;
    }

    let size: Coordinate = {x: 3, y: 3};
    let multiplier: Coordinate = {x: 3, y: 3};
    let numberOfFields: number;
    let fields: number[] = [];

    type Board = number[][][][];

    let board: Board = [];

    let numberOfSolutions: number;
    let numberOfPossibilities = 0;
    let possibilities: number[] = [];

    const randomNumberUpTo = (max: number): number => {
        return Math.floor(Math.random() * (max));
    }

    const max = (a: number, b: number): number => {
        if (a < b) {
            return b;
        } else {
            return a;
        }
    }

    const min = (a: number, b: number): number => {
        if (a < b) {
            return a;
        } else {
            return b;
        }
    }

    const swap = (array: number[], a: number, b: number) => {
        let temp = array[a];
        array[a] = array[b];
        array[b] = temp;
    }

    const updateNumberOfFields = (): number => {
        numberOfFields = (size.x * size.y) * (multiplier.x * multiplier.y);
        return numberOfFields;
    }

    const updateFields = () => {
        for (let i = 0; i < numberOfFields; i++) {
            fields[i] = i;
        }
    }

    const shuffleFields = () => {
        for (let i = 0; i < numberOfFields; i++) {
            let toSwap = randomNumberUpTo(numberOfFields);
            swap(fields, i, toSwap);
        }
    }

    const prepareFields = () => {
        updateNumberOfFields();
        updateFields();
        shuffleFields();
    }

    const updatePossibilities = () => {
        for (let i = 0; i < numberOfPossibilities; i++) {
            possibilities[i] = i;
        }
    }

    const shufflePossibilities = () => {
        for (let i = 0; i < numberOfPossibilities; i++) {
            let toSwap = randomNumberUpTo(numberOfPossibilities);
            swap(possibilities, i, toSwap);
        }
    }

    const getPrimaryCoords = (x: number, y: number): Coordinate => {
        let xR = Math.floor(x / multiplier['x']);
        let yR = Math.floor(y / multiplier['y']);

        return {x: xR, y: yR}
    }

    const getSecondaryCoords = (x: number, y: number): Coordinate => {
        let primaryCoords = getPrimaryCoords(x, y);
        
        let xR = x - (primaryCoords.x * multiplier['x']);
        let yR = y - (primaryCoords.y * multiplier['y']);

        return {x: xR, y: yR}
    }

    const getDetailedCoords = (x: number, y: number): {p: Coordinate, s: Coordinate} => {
        return {p: getPrimaryCoords(x, y), s: getSecondaryCoords(x, y)};
    }

    const checkUnsafeBoard = (p: Coordinate, s: Coordinate) => {
        if (board[p.x] == undefined) {
            board[p.x] = [];
        } 

        if (board[p.x][p.y] == undefined) {
            board[p.x][p.y] = [];
        }

        if (board[p.x][p.y][s.x] == undefined) {
            board[p.x][p.y][s.x] = [];
        }
    }

    const getBoard = (x: number, y: number): number => {
        let coords = getDetailedCoords(x, y);

        return board[coords.p.x][coords.p.y][coords.s.x][coords.s.y];
    }

    const setBoard = (x: number, y: number, set: number) => {
        let coords = getDetailedCoords(x, y);

        checkUnsafeBoard(coords.p, coords.s);

        board[coords.p.x][coords.p.y][coords.s.x][coords.s.y] = set;
    }

    const getBoxSize = (direction: 'x' | 'y'): number => {
        return (size[direction]);
    }

    const getFullSize = (direction: 'x' | 'y'): number => {
        return (size[direction] * multiplier[direction]);
    }

    const getMinFullSize = (): number => {
        return min(getFullSize('x'), getFullSize('y'));
    }

    const coordinteFromField = (i: number): Coordinate => {
        let x = Math.floor(i / getMinFullSize());
        let y = i - (x * getMinFullSize());

        return {x, y};
    }

    const updateNumberOfPossibilities = (): number => {
        numberOfPossibilities = max(getFullSize('x'), getFullSize('y'));
        return numberOfPossibilities;
    }

    const preparePossibilities = () => {
        updateNumberOfPossibilities();
        updatePossibilities();
        shufflePossibilities();
    }

    const clearBoard = () => {
        for (let x = 0; x < getFullSize('x'); x++) {
            for (let y = 0; y < getFullSize('y'); y++) {
                setBoard(x, y, 0);
            }
        }
    }

    const changeSize = (newSize: Coordinate): void => {
        size = newSize;
    }

    const changeMultiplier = (newMultiplier: Coordinate): void => {
        multiplier = newMultiplier;
    }

    const findEmptySlot = (): Coordinate => {
        for (let x = 0; x < getFullSize('x'); x++) {
            for (let y = 0; y < getFullSize('y'); y++) {
                if (getBoard(x, y) == 0) {
                    return {x, y};
                }
            }
        }

        return size;
    }

    const usedOn = (i: number, direction: 'x' | 'y', coord: number) => {
        for (let j = 0; j < getFullSize(direction); j++) {
            if (getBoard((direction === 'x' ? coord : j), (direction === 'y' ? coord : j)) == i) {
                return true;
            }
        }

        return false;
    };

    const usedOnBox = (x: number, y: number, i: number): boolean => {
        let primary = getPrimaryCoords(x, y);

        for (let i = 0; i < getBoxSize('x'); i++) {
            for (let j = 0; j < getBoxSize('y'); j++) {
                let xN = (primary.x * size['x']) + i;
                let yN = (primary.y * size['y']) + j;
                
                if (getBoard(xN, yN) == i) {
                    return true;
                }
            }
        }

        return false;
    }

    const isSafe = (x: number, y: number, i: number): boolean => {
        return !usedOn(i, 'x', x) && !usedOn(i, 'y', y) && !usedOnBox(x, y, i);
    }

    const solve = (): boolean => {
        let coord = findEmptySlot();

        if (coord == size) {
            return true;
        }

        for (let i = 0; i < numberOfFields; i++) {
            if (isSafe(coord.x, coord.y, possibilities[i])) {
                setBoard(coord.x, coord.y, possibilities[i]);

                if (solve()) {
                    return true;
                }

                setBoard(coord.x, coord.y, 0);
            }
        }

        return false;
    }

    const countSolutions = () => {
        let coord = findEmptySlot();

        if (coord == size) {
            numberOfSolutions++;
            return;
        }

        for (let i = 0; i < numberOfFields && numberOfSolutions < 2; i++) {
            if (isSafe(coord.x, coord.y, possibilities[i])) {
                setBoard(coord.x, coord.y, possibilities[i]);
                countSolutions();
            }

            setBoard(coord.x, coord.y, 0);
        }
    }

    const generatePuzzle = (goal: number) => {
        let emptyPlaces = 0;
        for (let i = 0; i < numberOfFields; i++) {
            let coord = coordinteFromField(fields[i]);
            let temp = getBoard(coord.x, coord.y);
            setBoard(coord.x, coord.y, 0);

            numberOfSolutions = 0;
            countSolutions();
            emptyPlaces++;

            if (numberOfSolutions != 1) {
                setBoard(coord.x, coord.y, temp);
                emptyPlaces--;
            }

            if (emptyPlaces == goal) {
                return;
            }
        }  
    }

    const preparePuzzle = () => {
        clearBoard();

        preparePossibilities();
        prepareFields();

        solve();
        generatePuzzle(80);
    }

    preparePuzzle();
    console.log(board);
    
</script>

<template>
    <mainview>
        <div v-for="px in multiplier['x']">
            <div v-for="py in multiplier['y']">
                <div v-for="sx in size['x']">
                    <div v-for="sy in size['y']">
                        {{board[px - 1][py - 1][sx - 1][sy - 1]}}
                    </div>
                </div>
            </div>
        </div>
    </mainview>
</template>

<style>

</style>