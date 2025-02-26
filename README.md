# C-_Final_Project_Diar
#include <iostream>
#include <vector>
#include <ctime>
#include <cstdlib>

using namespace std;

const int SIZE = 10;
const char WATER = '.';
const char SHIP = 'O';
const char HIT = 'X';
const char MISS = '*';

class Board {
public:
    vector<vector<char>> grid;
    vector<vector<char>> visibleGrid;

    Board() {
        grid.assign(SIZE, vector<char>(SIZE, WATER));
        visibleGrid.assign(SIZE, vector<char>(SIZE, WATER));
    }

    void placeShip(int size) {
        bool placed = false;
        while (!placed) {
            int x = rand() % SIZE;
            int y = rand() % SIZE;
            bool horizontal = rand() % 2;
            if (canPlace(x, y, size, horizontal)) {
                for (int i = 0; i < size; i++) {
                    if (horizontal)
                        grid[y][x + i] = SHIP;
                    else
                        grid[y + i][x] = SHIP;
                }
                placed = true;
            }
        }
    }

    void placeShips() {
        srand(time(0));
        placeShip(4);
        for (int i = 0; i < 2; i++) placeShip(3);
        for (int i = 0; i < 3; i++) placeShip(2);
        for (int i = 0; i < 4; i++) placeShip(1);
    }

    bool canPlace(int x, int y, int size, bool horizontal) {
        if (horizontal && x + size > SIZE) return false;
        if (!horizontal && y + size > SIZE) return false;
        for (int i = 0; i < size; i++) {
            if (horizontal && grid[y][x + i] != WATER) return false;
            if (!horizontal && grid[y + i][x] != WATER) return false;
        }
        return true;
    }

    void printBoard(bool showShips) {
        cout << "  0123456789" << endl;
        for (int i = 0; i < SIZE; i++) {
            cout << i << " ";
            for (int j = 0; j < SIZE; j++) {
                cout << (showShips ? grid[i][j] : visibleGrid[i][j]) << " ";
            }
            cout << endl;
        }
    }

    bool attack(int x, int y) {
        if (x < 0  x >= SIZE  y < 0 || y >= SIZE) return false;
        if (grid[y][x] == SHIP) {
            grid[y][x] = HIT;
            visibleGrid[y][x] = HIT;
            return true;
        }
        visibleGrid[y][x] = MISS;
        return false;
    }

    bool hasShips() const {
        for (const auto& row : grid)
            for (char cell : row)
                if (cell == SHIP) return true;
        return false;
    }
};

class Player {
public:
    virtual bool makeMove(Board &enemyBoard) = 0;
};

class HumanPlayer : public Player {
public:
    bool makeMove(Board &enemyBoard) override {
        int x, y;
        cout << "Введите координаты (x y): ";
        cin >> x >> y;
        return enemyBoard.attack(x, y);
    }
};

class AIPlayer : public Player {
public:
    bool makeMove(Board &enemyBoard) override {
        int x, y;
        do {
            x = rand() % SIZE;
            y = rand() % SIZE;
        } while (enemyBoard.visibleGrid[y][x] != WATER);
        
        cout << "Компьютер стреляет в: " << x << " " << y << endl;
        return enemyBoard.attack(x, y);
    }
};

class Game {
public:
    Board playerBoard, aiBoard;
    HumanPlayer player;
    AIPlayer ai;

    Game() {
        playerBoard.placeShips();
        aiBoard.placeShips();
    }

    void play() {
        while (playerBoard.hasShips() && aiBoard.hasShips()) {
            cout << "Ваше поле:" << endl;
            playerBoard.printBoard(true);
            cout << "Поле противника:" << endl;
            aiBoard.printBoard(false);
            
            cout << "Ваш ход:" << endl;
            player.makeMove(aiBoard);
            
            if (!aiBoard.hasShips()) {
                cout << "Вы победили!" << endl;
                return;
            }
            
            cout << "Ход компьютера:" << endl;
            ai.makeMove(playerBoard);
            
            if (!playerBoard.hasShips()) {
                cout << "Вы проиграли!" << endl;
                return;
            }
        }
    }
};

int main() {
    setlocale(LC_ALL, "Russian");
    Game game;
    game.play();
    return 0;
}
