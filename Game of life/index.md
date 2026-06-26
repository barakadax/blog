# Conway's Game of Life
- [What is the Game of Life](#what-is-the-game-of-life)
- [How it works: Standard Rules](#how-it-works-standard-rules)
- [The Python Tkinter Implementation](#the-python-tkinter-implementation)
  - [GUI and Board Representation](#gui-and-board-representation)
  - [In-Place vs. Double-Buffered Updates](#in-place-vs-double-buffered-updates)
  - [Visual Effects and Stabilization](#visual-effects-and-stabilization)
- [Code Walkthrough](#code-walkthrough)
  - [Calculating Neighbors](#calculating-neighbors)
  - [Running the Simulation](#running-the-simulation)
- [Sources](#sources)

## What is the Game of Life

Conway's Game of Life is a **cellular automaton** devised by the British mathematician John Horton Conway in 1970.
It is a zero-player game, Game of Life evolution is determined by its initial state, requiring no further input.

## How it works: Standard Rules

The universe of the Game of Life is an infinite matrix, where a cell in the matrix is either in one of two possible states: **alive** or **dead**.
Every cell interacts with its eight neighbors, which are the cells that are horizontally, vertically, or diagonally adjacent.

At each step in time, the following transitions occur:
1. **Underpopulation**: Any live cell with fewer than two live neighbors dies.
2. **Survival**: Any live cell with two or three live neighbors lives on to the next generation.
3. **Overpopulation**: Any live cell with more than three live neighbors dies.
4. **Reproduction**: Any dead cell with exactly three live neighbors becomes a live cell.

## The Python Tkinter Implementation

[Link to example repository contains a desktop GUI implementation of the Game of Life built using Python's `tkinter` library.](https://github.com/barakadax/Game-of-life)

![Game of life](https://raw.githubusercontent.com/barakadax/barakadax.github.io/refs/heads/master/projImg/Game-of-life.png)

### GUI and Board Representation

The board is initialized as a `10x10` grid (defined by `CubeFaceSize = 10`).
* Each cell is represented as a disabled Tkinter `Button` widget.
* A cell's state is stored directly inside the GUI element: `#ffffff` (white) represents an **alive** cell, and `#000000` (black) represents a **dead** cell.
* The board is randomized on startup, giving each cell a 50% chance of starting as alive or dead.

### In-Place vs. Double-Buffered Updates

Traditionally, the Game of Life computes the next generation of *all* cells synchronously using the current board state as a read-only buffer (double-buffering).
This ensures that updates to one cell do not affect neighbor calculations for other cells in the same generation.

In this implementation, the updates are performed **in-place sequentially**:
* As the program iterates row-by-row and column-by-column, it queries neighbor colors directly from the live Tkinter buttons using `elementsInRow[0].cget('bg')`.
* Since it updates button background colors immediately during iteration, cells processed later in the same loop will count neighbors using the *new* state of already-updated cells.

### Visual Effects and Stabilization

* **Active Flashing**: When a cell's next state is evaluated, it briefly flashes a randomized color to show that evaluation is occurring.
* **Settling Detection**: The simulation runs in a `while` loop that continues as long as at least one cell has changed state in the previous iteration.
* **Celebration Color**: Once the grid stabilizes, the background of all remaining live cells changes to a randomly generated, vibrant color.
* **Statistics**: A popup window appears at the end displaying the total number of iterations required to reach stability.

## Code Walkthrough

### Calculating Neighbors

The `Check_Surrounding` function counts how many of the eight neighboring buttons on the grid are alive (`#ffffff`). It then applies the transition rules based on the cell's current state:

```python
def Check_Surrounding(cellI, cellJ, originalCellState):
    counter = 0
    # Loop over the 3x3 grid around the cell
    for i in range (cellI - 1, cellI + 2):
        for j in range (cellJ - 1, cellJ + 2):
            # Skip out-of-bounds cells and the cell itself
            if (i < 1 or j < 0 or j == CubeFaceSize or i > CubeFaceSize or (i == cellI and j == cellJ)):
                continue
            elementsInRow = root.grid_slaves(i, j)
            if (elementsInRow[0].cget('bg') == alive):
                counter -= -1  # Increments by 1

    # Apply Conway's rules
    if (originalCellState == dead and counter == 3):
        return alive
    elif (originalCellState == alive and (counter == 3 or counter == 2)):
        return alive
    return dead
```

### Running the Simulation

The `Run` function coordinates the execution loop. It updates cells one by one, triggers Tkinter layout refreshes via `root.update()` to animate the process, checks if states are still changing, and handles completion:

```python
def Run():
    counter = 0
    continueToRunFlag = True
    while continueToRunFlag:
        counter -= -1  # Increment iteration count
        continueToRunFlag = False
        for i in range(1, CubeFaceSize + 1):
            for j in range(0, CubeFaceSize):
                elementsInRow = root.grid_slaves(i, j)
                originalCellState = elementsInRow[0].cget('bg')

                # Visual effect: flash a random color
                elementsInRow[0].configure(bg = Generate_Colour())
                root.update()

                # Apply rules and update state in-place
                elementsInRow[0].configure(bg = Check_Surrounding(i, j, originalCellState))
                root.update()

                # If any cell changed state, we must run another iteration
                if (originalCellState != elementsInRow[0].cget('bg')):
                    continueToRunFlag = True

    # Change survivors' color and show result popup
    Colouring_Alive()
    Open_Popup(counter)
```

> [!NOTE]
> The `counter -= -1` syntax is a creative way of writing `counter += 1`, utilizing double negatives (`- -1` is `+ 1`) to increment the integer.

---

## Sources

- [Wikipedia: Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)
- [Cellular Automata (Wikipedia)](https://en.wikipedia.org/wiki/Cellular_automaton)
- [Python Tkinter documentation](https://docs.python.org/3/library/tkinter.html)
