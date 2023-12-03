---
title: C++ Game of Life
author: Brunon
date: 2022-11-09 12:00:00 +0100
categories: [Brunon]
tags: [University, Code]
pin: false
math: true
mermaid: true
render_with_liquid: false
#comment
---
C++ project I made with my friend Szczepan Letkiewicz during our C++ course at University of Twente.

### Main engine of the simulation

```c++
// During the process of compilation, the variable is changed to an asterix "*"
#define LIVING_CELL '*'
#define DEAD_CELL ' '
#define UNDEF_CELL '?'

class Game {
private:
    //These variables can only be called inside of this class
    //Pointer for 1 byte element of matrix
    //Reserves a certain amount of memory for the matrix. This amount is decided by the size (user input)
    char* matrix;
    //Set new value for the next cells, if cell value is outside of memoery programme ends.
    void setCellValue(int, int, char);
    // Copies the values in first matrix to a matrix that is then displayed
    void syncMatrix(void);
    //Reads the value in the matrix
    char getCellValue(int, int);
    //Returns the number of living cells around a cell.
    int getNumberOfLivingNeighbours(int, int);
    //sets cell to alive
    void reviveCell(int, int);
    //sets cell to dead
    void killCell(int, int);

protected:
    //Variables that can be called by this class and other classes that inherate this one
    int rows, cols, round=0, n_living_cells=0;
    char* matrix_to_display;

public:
    //Can be called by everything
    Game(int, int);
    //Destructor
    ~Game();
    //clears matrix
    void clearMatrix(void);
    //Generates living cells in matrix randomly
    void generateMatrix(void);
    // Decides which cells live and which die
    void makeStep(void);
};
//Constructor, Creates a matrix using the rows and colums decided by player
Game::Game(int rows_, int cols_) {
    cols = cols_;
    rows = rows_;
    matrix_to_display = new char[rows * cols];
    matrix = new char[rows * cols];
    generateMatrix();
};

Game::~Game() {
    delete[] matrix_to_display;
    delete[] matrix;
}

void Game::generateMatrix(void) {
    clearMatrix();
    srand(time(0));
    for (int r=0; r < rows; r++) {
        for (int c=0; c < cols; c++) {
            //checks if the random value is odd or even (1 or 0).
            if (rand() % 2 == 1) {
                //if option is 1, set cell status to living
                *(matrix + r * cols + c) = LIVING_CELL;
                //incerase cell counter
                n_living_cells++;
            }
        }
    }
    syncMatrix();
};

void Game::clearMatrix(void) {
    //For every cell in the matrix
    for (int r=0; r < rows; r++) {
        for (int c=0; c < cols; c++) {
            //Kills every cell in both matrixes
            *(matrix_to_display + r * cols + c) = DEAD_CELL;
            *(matrix + r * cols + c) = DEAD_CELL;
        }
    }
};

void Game::makeStep(void) {
    int n_living_neighbours, cell_value;
    for (int r=0; r < rows; r++) {
        for (int c=0; c < cols; c++) {
            n_living_neighbours = getNumberOfLivingNeighbours(r,c);
            cell_value = getCellValue(r,c);
            //If Cell is alive and if cell has 2 or 3 neighbours continue
            if (cell_value == LIVING_CELL) {
                if (n_living_neighbours == 2 || n_living_neighbours == 3) {
                    continue;
                }
            } else  {
                //of cell has 2 neighbours revive cell
                if (n_living_neighbours == 3) {
                    reviveCell(r, c);
                    continue;
                }
            }
            //kill
            killCell(r, c);
        }
    }
    //increment round counter
    round++;
    //sync
    syncMatrix();
};

void Game::syncMatrix(void) {
    //For every row and columm
    for (int r=0; r < rows; r++) {
        for (int c=0; c < cols; c++) {
            //copies a value
            *(matrix_to_display + r * cols + c) = *(matrix + r * cols + c);
        }
    }
};

int Game::getNumberOfLivingNeighbours(int row, int col) {
    int n_living_neighbours=0;
    for (int r=-1; r < 2; r++) {
        for (int c=-1; c < 2; c++) {
            //The current cell is omited, because it is not a neighbour
            if (r == 0 && c == 0) continue;
            //checks if the cell is alive
            if (getCellValue(row + r, col + c) == LIVING_CELL) {
                //if so it adds to the living neighbours
                n_living_neighbours++;
            }
        }
    }
    //returns the amount of living neighbours per given Cell.
    return n_living_neighbours;
};

void Game::setCellValue(int row, int col, char val) {
    if (row < 0 || row >= rows) {
        //Excpetion is made, if the above criteria is met. The programm ends and this message is printed.
        throw std::invalid_argument("The provided row number is out of the range");
    }
    if (col < 0 || col >= cols) {
        throw std::invalid_argument("The provided column number is out of the range");
    }

    //Checks value of cell, if the value is not equal to the new value, adjust the live cell counter
    if (*(matrix + row * cols + col) != val) {
        //if the following cell conditions are met, adjust the live cell counter.
        switch (val) {
            case LIVING_CELL:
                //Increase live celll counter on top of page
                n_living_cells++;
                //need to break, because there are 2 cases
                break;
            case DEAD_CELL:
                //decrease live celll counter on top of page
                n_living_cells--;
                break;
        }
        //Sets new value for the cell.
        *(matrix + row * cols + col) = val;
    }
};

char Game::getCellValue(int row, int col) {
    //If the value is outside of matrix, lable as undefined
    if (row < 0 || row >= rows) {
        return UNDEF_CELL;
    }
    if (col < 0 || col >= cols) {
        return UNDEF_CELL;
    }
    //returns the value of a cell indexed by row and col
    return *(matrix_to_display + row * cols + col);
};

void Game::reviveCell(int row, int col) {
    //sets cell to alive
    setCellValue(row, col, LIVING_CELL);
};

void Game::killCell(int row, int col) {
    //sets cell to dead
    setCellValue(row, col, DEAD_CELL);
};


```

Rest of the code can be found on my github: <https://github.com/MessedUpCat/Just-another-game-of-life/tree/main>