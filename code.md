//--------------------------------------------------
//snake in terminal by Rajas Sharma - designed for windows 
// REPLACE conio.h WITH MORE GENERAL SOLUTION
//--------------------------------------------------
#include <iostream>
#include <string>//string manipulation
#include <thread>//to generate a framerate - to sleep the thread
#include <chrono>//std::chrono::miliseconds(time);
#include <conio.h>//_getch and _kbhit -- NEED TO REPLACE THIS WITH MORE GENERAL SOLUTION - DOESNT WORK IN LINUX
#include <vector>
//#include <cstdlib>

//initialise world variables --> parameters to change before compile time
const int wallx = 28, wally = 28;
const int framerate = 12;//game speed -> fps
const std::string defaulttile = "..", snaketile = "[]", appletile = "||";
std::string worldtile = "";
std::string bufferline;//string to help seperate frames - print in one go to keep frame transisiton smooth

int gamearray[wallx][wally] = {0};
bool gamestate = true;
bool appletilecurrent = true;
bool eat = false;
//1up 2right 3down 4left -> what each dir represents
int dir = 3;

std::vector<int> snakex;
std::vector<int> snakey;
std::vector<int> snakexhold;
std::vector<int> snakeyhold;
int x=wallx/2, y=wally/2;
int xA= wallx / 2, yA= 3 * wally / 4;

//for for loops
int i, j;

//initialise snake vectors
void init() {
	gamearray[x][y] = 1; gamearray[xA][yA]=2;
	snakex.push_back(x);snakey.push_back(y);
	snakexhold.push_back(x); snakeyhold.push_back(y);
	for (i = 0; i < wallx; i++) { bufferline += "  "; }
}
//reset the screen - set game array to all 0s and apple 2 - clear snake from game array
void reset() {
	for (j = 0; j < wally; j++) {
		for (i = 0; i < wallx; i++) {
			gamearray[i][j] = 0;
		}
	}
	gamearray[xA][yA]=2;
}
//checks for user input and changes direction accordingly
void checkuserinput() {
	char c = tolower(_getch());
	if (c == 'w'&&dir!=3) { dir = 1; }
	if (c == 'd'&&dir!=4) { dir = 2; }
	if (c == 's'&&dir!=1) { dir = 3; }
	if (c == 'a'&&dir!=2) { dir = 4; }
}
//check if snake is colliding with self
void checkdeath() {
	for (i = 0; i < snakex.size(); i++) {
		for (j = 0; j < snakex.size();j++) {
			if (snakex[i] == snakex[j] && snakey[i] == snakey[j] && i != j&&!eat) {
				gamestate = false;
			}
		}
	}
}
//sets snake coords to a temp array and move each coord one space to the back of the array
void updatesnakebody() {
	for (i = 0; i < snakex.size();i++) {
		snakexhold[i] = snakex[i];
		snakeyhold[i] = snakey[i];
	}
	for (i = 1; i < snakex.size(); i++) {
		snakey[i] = snakeyhold[i - 1];
		snakex[i] = snakexhold[i - 1];
	}
}
//imbbeds snake coords into world
void setgamearray() {
	for (i = 0; i < snakex.size(); i++) {
		gamearray[snakex[i]][snakey[i]] = 1;
	}
}
//reset screen - move snake - set snake coords - all depending on dir
void updatehead() {
	switch (dir) {
		//1up 2right 3down 4left
		case 1:
			reset();updatesnakebody();
			y -= 1;snakey[0] = y;
			setgamearray();break;
		case 2:
			reset();updatesnakebody();
			x += 1;snakex[0] = x;
			setgamearray();break;
		case 3:
			reset();updatesnakebody();
			y += 1;snakey[0] = y;
			setgamearray();break;
		case 4:
			reset();updatesnakebody();
			x -= 1;snakex[0] = x;
			setgamearray();break;
	}
	//check wall collision
	if (x >= wallx) { gamestate = false; }
	if (y >= wally) { gamestate = false; }
	if (x <= 0) { gamestate = false; }
	if (y <= 0) { gamestate = false; }
}
//set random position for apple when eaten - DOESNT CHECK FOR SNAKE COLLISION
void generateapple() {
	xA = rand() % wallx; yA = rand() % wally;
	gamearray[xA][yA] = 2;
	eat = true;
}
//rendering algorithm
//reads gamearray for world information and appends 
// to a string that is printed in one pass at end of function
void render() {
	worldtile = "";
	appletilecurrent = false;
	for (i = 0; i < wally; i++) {
		for (j = 0; j < wallx; j++) {
			switch (gamearray[j][i]) {
			case 0:
				worldtile += defaulttile;
				break;
			case 1:
				worldtile += snaketile;
				break;
			case 2:
				worldtile += appletile;
				appletilecurrent = true;
				break;
			}
		}
		worldtile += '\n';
	}
	std::cout << worldtile;
	//check if apple has been rendered - 
	//if not then it has been eaten and needs to be regenrated
	if (!appletilecurrent) { 
		generateapple(); 
		snakex.push_back(x); snakey.push_back(y); 
		snakexhold.push_back(x); snakeyhold.push_back(y);
	}
}

//----------------------------------------------------------------------------------------------------------
//main function:
//----------------------------------------------------------------------------------------------------------

int main() {
	init();
	while (gamestate) {
		if (_kbhit()) {
			checkuserinput();
		}
		std::srand(std::time(0));//seed random function in current time
		updatehead();
		render();
		checkdeath();
		std::this_thread::sleep_for(std::chrono::milliseconds(1000 / framerate));//sleeps thread according on framerate
		std::cout << bufferline << '\n';//creates a new line for a gap between frames
		eat = false;
	}
	for (i = 0; i < 100000; i++) { std::cout << "score:" << snakex.size() - 1; }
	return 0;
}
