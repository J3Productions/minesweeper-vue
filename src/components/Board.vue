<template>
  <div>
    <div class="row" v-if="!gameOver && !gameWon && timer >= 1"><h1>Right Click to Set Flag</h1></div>
    <div class="row" v-if="!gameOver && !gameWon && timer >= 1"><span>Time left: {{ timer }}</span></div>
    <p>{{ errorMsg }}</p>
    <br v-if="!gameOver && timer >= 1">



    <div class="board-row" v-for="(row, y) in this.board" v-bind:key="y" v-if="!gameOver && !gameWon && timer >= 1">
        <cell
        :x="x"
        :y="y"
        :value = "board[y][x].value"
        :bomb = "board[y][x].bomb"
        :playerName ="board.name"
        :isDisplayingValue = "board[y][x].isDisplayingValue"
        :isRevealed = "board[y][x].isRevealed"
        :getCanFlag = "getCanFlag"
        @cell-click="cellClick"
        @clickedOnBomb="gameOver = true"
        @flag="onFlag"
        v-for="(cell, x) in row"
        :key="x"></cell>
    </div>


    <div v-if="!gameOver && !gameWon && timer >= 1">
      <h6><button @click="goToCheatMode" class="button">Cheat Mode</button></h6>
    </div>


    <div v-if="(gameOver || timer < 1) && !gameWon">
      <h1>Boom! You're Dead!</h1>
      <div class="row" v-if="gameOver"><span>You clicked on a mine!</span></div>
      <div class="row" v-if="timer < 1 && !gameOver"><span>You ran out of time!</span></div>
      <h6><button @click="goToMenu" class="button">Menu</button></h6>
      <h6><button @click="restartGame" class="button">Restart Game</button></h6>
    </div>
    
    <div v-if="gameWon">
      <h1>You Won! Great Work!</h1>
      <div class="row"><span>You finished with {{ score }} seconds to spare!</span></div>
      <h6><button @click="goToMenu" class="button">Menu</button></h6>
      <h6><button @click="restartGame" class="button">New Game</button></h6>

      <div class="board">
        <h3 style="color: white">Top 5 Scores!</h3>
        <table class="scoreBoard" border ="1" align="center">
          <tr>
            <th>Rank</th>
            <th>Name</th>
            <th>Board Size</th>
            <th>Clear Time</th>
          </tr>
          <tr v-if="scoreArr[0] !== null">
            <td>1</td>
            <td>{{ scoreArr[0].name }}</td>
            <td>{{ scoreArr[0].size }}</td>
            <td>{{ scoreArr[0].time }} sec</td>
          </tr>
          <tr v-if="scoreArr[1] !== null">
            <td>2</td>
            <td>{{ scoreArr[1].name }}</td>
            <td>{{ scoreArr[1].size }}</td>
            <td>{{ scoreArr[1].time }} sec</td>
          </tr>
          <tr v-if="scoreArr[2] !== null">
            <td>3</td>
            <td>{{ scoreArr[2].name }}</td>
            <td>{{ scoreArr[2].size }}</td>
            <td>{{ scoreArr[2].time }} sec</td>
          </tr>
          <tr v-if="scoreArr[3] !== null">
            <td>4</td>
            <td>{{ scoreArr[3].name }}</td>
            <td>{{ scoreArr[3].size }}</td>
            <td>{{ scoreArr[3].time }} sec</td>
          </tr>
          <tr v-if="scoreArr[4] !== null">
            <td>5</td>
            <td>{{ scoreArr[4].name }}</td>
            <td>{{ scoreArr[4].size }}</td>
            <td>{{ scoreArr[4].time }} sec</td>
          </tr>
        </table>
        <h6><button @click="resetBoard" class="button">Reset High Scores</button></h6>
      </div>
    </div>

  </div>
</template>

<script lang="ts">
  import { Component, Vue, Prop } from "vue-property-decorator";
  import Cell from "./Cell.vue";

  @Component({
  components: {
  Cell
  }
  })
  export default class Board extends Vue {
  /**
  * The cells in the game board
  */
  private board: Cell[][] = [];

  /**
  * Indicates whether the game has been finished with a loss
  */
  private gameOver = false;

  /**
  * Indicates whether the game has been finished with a win
  */
  private gameWon = false;

  /**
  * Indicate the scoreboard shows or not.
  */
  private swBoard = false;

  /**
  * Indicates whether the cheat mode in on
  */
  private CheatOn = false;

  /**
  * The size of the board in the x dimension
  */
  @Prop() private xSize!: any;

  /**
  * The size of the board in the y dimension
  */
  @Prop() private ySize!: any;

  /**
  * The number of bombs contained in the board
  */
  @Prop() private numBombs!: any;

  /**
  * Player name.
  */
  @Prop() private playerName!: any;

  /**
  * The error message to display to the user, if applicable
  */
  private errorMsg: string = '';

  /**
  * The number of flags currently placed on the board
  */
  private flagCount: number = 0;

  /**
  * The time taken to complete the game/the time left to complete the game
  */
  private timer: number = 0;

  /**
  * The final time taken to clear the board
  */
  private score: number = 0;

  /**
  * Array save the top 5 score.
  */
  private scoreArr: any[] = [];

  /**
  * Clears all scores from memory
  */
  public resetBoard() {
      if(typeof(Storage) !== "undefined") {
          localStorage.clear();
          this.scoreArr = [null, null, null, null, null];
          console.log("Score board clear!");
      }
      else {
          console.log("Sorry, your browser does not support local storage...");
      }
  }

  /**
   * Retrieves all the scores for the number of mines selected and inserts the current score into the list of scores, storing the current high score list in scoreArr
   */
  public sendScore() {
      if(typeof(Storage) !== "undefined") {
          //String that is used for the key to store the score under
          let keyString = "mines" + this.numBombs + "rank";

          //Array that retrieves all the scores in JSON format
          let scoreJSONArr = [];

          //Array that stores all the scores in object format
          let scoreObjArr = [];

          for (let i = 0; i < 5; i++) {
              scoreJSONArr[i] = localStorage.getItem(keyString.concat(i.toString()));
              // @ts-ignore
              scoreObjArr[i] = JSON.parse(scoreJSONArr[i]);
          }

          let scoreObj = {
              name: this.playerName,
              size: this.xSize * this.ySize,
              mines: this.numBombs,
              time: (3 * (this.xSize * this.ySize)) - this.score,
          };

          for (let i = 0; i < 5; i++) {
              if (scoreObjArr[i] == null || scoreObj.time < scoreObjArr[i].time) {
                  scoreObjArr.splice(i, 0, scoreObj);
                  break;
              }
          }

          for (let i = 0; i < 5; i++) {
              if (scoreObjArr[i] == null) {
                  scoreJSONArr[i] = null;
              }
              else {
                  scoreJSONArr[i] = JSON.stringify(scoreObjArr[i]);
              }
              // @ts-ignore
              localStorage.setItem(keyString.concat(i.toString()), scoreJSONArr[i]);
          }
          this.scoreArr = scoreObjArr;
      }
      else {
          console.log("Sorry, your browser does not support local storage...");
      }

  }

  /**
  * Called when the board is created
  */
  public created() {
    console.log('creating board')
    var board = this.initializeCells([], this.xSize, this.ySize);
    Object.seal(this.board);
    board = this.genBombs(board, this.numBombs);
    this.board = board;
    this.computeValues();
    //Here's where a formula for calculating the time left would go if we do the countdown timer path
    this.timer = 3 * (this.xSize * this.ySize);
    setInterval(() => this.timer--, 1000);
  }

  /**
  * Called when the game ends in a loss
  */
  private onGameOver(){
    this.gameOver = true;
    this.errorMsg = '';
  }

  /**
  * Reinitializes the game with a new game board using the same parameters
  */
  restartGame(){
    this.gameOver = false;
    this.gameWon = false;
    this.CheatOn = false;
    this.swBoard = false;
    var board = this.initializeCells([], this.xSize, this.ySize);
    Object.seal(this.board);
    board = this.genBombs(board, this.numBombs);
    this.board = board;
    this.computeValues();
    this.flagCount = 0;
    this.errorMsg = '';
    this.timer = 3 * (this.xSize * this.ySize);
  }

  /**
  * Emits the 'goToMenu' event, indicating that the UI should return to the input menu
  */
  goToMenu(){
    this.gameOver = false
    this.CheatOn = false
    this.swBoard = false;
    this.$emit('goToMenu')
  }

  /**
  * The function open or close the cheat mode.
  */
  goToCheatMode() {
    if(this.CheatOn == false)
    {
    this.CheatOn = true;
    var cheatBoard = this.cheatModeOn(this.board, this.xSize, this.ySize);
    Object.seal(this.board);
    this.board = cheatBoard;
    }
    else if(this.CheatOn == true)
    {
    this.CheatOn = false;
    var cheatBoard = this.cheatModeOff(this.board, this.xSize, this.ySize);
    Object.seal(this.board);
    this.board = cheatBoard;
  }

  }

  /**
  * Reveals the values of the proper cells starting from a given initial point
  * @param initX The x coordinate of the initial point
  * @param initY The y coordinate of the initial point
  */
  public recSearch(initX: number, initY: number){
    this.recHelper(initX, initY);
  }

  /**
  * Gets the number of bombs that are adjacent to a cell, or -1 if the cell is a bomb
  * @param xPos The x coordinate of the cell to use
  * @param yPos The y coordinate of the cell to use
  * @returns The number of bombs adjacent to the cell at the given coordinates
  */
  private checkAdjacent(xPos: number, yPos: number): number {
    let count = 0;
    if(this.isInBoard(xPos + 1, yPos + 1) && this.board[yPos + 1][xPos + 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos + 1, yPos) && this.board[yPos][xPos + 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos, yPos + 1) && this.board[yPos + 1][xPos].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos - 1, yPos - 1) && this.board[yPos - 1][xPos - 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos - 1, yPos) && this.board[yPos][xPos - 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos, yPos - 1) && this.board[yPos - 1][xPos].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos - 1, yPos + 1) && this.board[yPos + 1][xPos - 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos + 1, yPos - 1) && this.board[yPos - 1][xPos + 1].getBomb()){
      count = count + 1;
    }
    if(this.isInBoard(xPos, yPos) && this.board[yPos][xPos].getBomb()){
      return(-1);
    }
    return(count);
  }

  /**
   * Reveals the values of the proper cells starting from a given initial point
   * @param xPos The x coordinate of the initial point
   * @param yPos The y coordinate of the initial point
   */
  private recHelper(xPos: number, yPos: number) {
    if(xPos < 0 || xPos >= this.xSize || yPos < 0 || yPos >= this.ySize){
      return;
    }

    let value = this.board[yPos][xPos].value;
    if(value >= 0) {
      if(this.board[yPos][xPos].isFlag) {
        return;
      }
      if(value > 0 || this.board[yPos][xPos].isDisplayingValue){
        this.board[yPos][xPos].displayValue();
        return;
      }
      else{
        this.board[yPos][xPos].displayValue();
        this.recHelper(xPos + 1, yPos);
        this.recHelper(xPos - 1, yPos);
        this.recHelper(xPos, yPos + 1);
        this.recHelper(xPos, yPos - 1);
        this.recHelper(xPos + 1, yPos + 1);
        this.recHelper(xPos - 1, yPos - 1);
        this.recHelper(xPos + 1, yPos - 1);
        this.recHelper(xPos - 1, yPos + 1);
      }
    }
  }

  /**
   * Initializes the Cells in a given board array with default values
   * @param board The board in which to initialize the cells
   * @param xSize The size of the board in the x dimension
   * @param ySize The size of the board in the y dimension
   * @returns A board of the given size containing default cells
   */
  private initializeCells(
    board: Cell[][],
    xSize: number,
    ySize: number
  ): Cell[][] {
    for (let i = 0; i < ySize; i++) {
      board.push(new Array(ySize));
      for (let j = 0; j < xSize; j++) {
        board[i][j] = new Cell();
        board[i][j].x = j;
        board[i][j].y = i;
        board[i][j].isDisplayingValue = false;
        board[i][j].isFlag = false;
        //This variable is for Cheat mode.
        board[i][j].isRevealed = false;
      }
    }
    return board;
  }




   /**
   * Revealed all space to become cheat mode
   * @param board The board in which is playing
   * @param xSize The size of the board in the x dimension
   * @param ySize The size of the board in the y dimension
   * @returns A board of the given size containing open cells
   */
  private cheatModeOn(
    board: Cell[][],
    xSize: number,
    ySize: number
  ): Cell[][] {
    for (let i = 0; i < ySize; i++) {
      for (let j = 0; j < xSize; j++) {
        board[i][j].cheatOn();
      }
    }
    return board;
  }


   /**
   * Close all the unrevealed spave to quit cheat mode
   * @param board The board in which is playing.
   * @param xSize The size of the board in the x dimension
   * @param ySize The size of the board in the y dimension
   * @returns A board of the given size containing original cells
   */
  private cheatModeOff(
    board: Cell[][],
    xSize: number,
    ySize: number
  ): Cell[][] {
    for (let i = 0; i < ySize; i++) {
      for (let j = 0; j < xSize; j++) {
        board[i][j].cheatOff();
      }
    }
    return board;
  }







  /**
   * Computes the values associated with all cells in the game board
   */
  private computeValues() {
    for(let i = 0; i < this.ySize; i++) {
      for(let j = 0; j < this.xSize; j++) {
        this.board[i][j].value = this.checkAdjacent(j, i);
      }
    }
  }

  /**
   * Called when the 'cell-click' event is emitted from a cell
   * @param coord The coordinates, in the format { x: x, y: y }, of the clicked cell
   */
  private cellClick(coord: any) {
    this.recSearch(coord.x, coord.y);
  }

  /**
   * Called when the 'flag' event is emitted from a cell
   * @param coord The coordinates, in the format { x: x, y: y } of the clicked cell
   */
  private onFlag(coord: any) {
    this.errorMsg = '';
    if(this.board[coord.y][coord.x].isFlag){
      this.flagCount = this.flagCount - 1;
    }
    else{
      this.flagCount = this.flagCount + 1;

    }
    this.board[coord.y][coord.x].isFlag = !this.board[coord.y][coord.x].isFlag;
    if(this.checkAllFlagged()){
      this.gameWon = true;
      this.board[coord.y][coord.x].isFlag = false;
      this.flagCount = 0;
      this.sendScore();
    }


  }

  /**
   * Gets a value indicating whether the user can currently place a flag
   * @param unFlag Indicates whether the user is removing or adding a flag
   */
  private getCanFlag(): boolean{
    if(this.flagCount + 1 <= this.numBombs){
      return(true);
    }
    else{
      this.errorMsg = 'Number of Flags cannot exceed number of Bombs';
      return(false);
    }
  }

  /**
   * Checks if all bombs have been correctly flagged
   * @returns A value indicating whether all bombs have been correctly flagged
   */
  private checkAllFlagged(): boolean {
    let correct = 0;
    for(let y = 0; y < this.ySize; y++){
      for(let x = 0; x < this.xSize; x++){
        if(this.board[y][x].getFlag() && this.board[y][x].getBomb()){
          correct = correct + 1;
        }
      }
    }
    if(correct == this.numBombs){
      this.score = this.timer;
      return(true);
    }
    else{

      return(false);
    }
  }

  /**
   * Places a given number of bombs at random locations on the given board
   * @param board The board in which to place bombs
   * @param numBombs The number of bombs to place
   * @returns A board with numBombs of its cells containing bombs
   */
  private genBombs(board: Cell[][], numBombs: number): Cell[][] {
    let genBombCounter = 0;

    while (genBombCounter < numBombs) {
      const xBomb = Math.floor(Math.random() * this.xSize);
      const yBomb = Math.floor(Math.random() * this.ySize);

      if (!board[yBomb][xBomb].getBomb()) {
        board[yBomb][xBomb].setBomb(true);
        genBombCounter = genBombCounter + 1;
      }
    }

    return board;
  }

  /**
   * Gets a value indicating whether the given (x, y) position is within the board
   * @param x The x position
   * @param y The y position
   * @returns A value indicating whether the given (x, y) position is within the board
   */
  private isInBoard(x: number, y: number): boolean {
    return this.board[y] !== undefined && this.board[y][x] !== undefined;
  }
}
</script>
<style scoped>
table {
  border-spacing: 10px;
}
.board-row {
  margin-bottom: -3px;
}
.cell {
  width: 15px;
  height: 15px;
  background-color: white;
  border: 1px solid black;
  display: inline-block;
}
.cell-bomb {
  width: 15px;
  height: 15px;
  background-color: black;
  border: 1px solid white;
  display: inline-block;
}
.cell-flag {
  width: 15px;
  height: 15px;
  background-color: green;
  border: 1px solid white;
  display: inline-block;
}
.button {
    background-color: white; /* Green */
    border: none;
    color: black;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
}
div.board
{
  color: black;
}
table.scoreBoard
{
    background-color: white;
}
</style>
