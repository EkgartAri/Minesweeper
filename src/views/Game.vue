<template>
    <div id="game" class="game-page">
        <router-link to="/" class="game-page__go-back">Назад</router-link>
        <h2>{{result}}</h2>
        <div class="game-page__row">
            <div v-for="(row, i) in cells" :key="i">
                <div v-for="(cell, j) in row" :key="j" 
                        :class="['game-page__cell', cell.isClear ? 'clear' : '']" 
                        @click="filp(i, j, cell)"
                        @contextmenu="mark(cell, $event)">
                {{ showCell(cell) }}
                </div>
            </div>
        </div>
    <div style="margin-top: 40px;">
      <div>
        <label>Размер: </label>
        <input type="number" v-model="rowCount">
      </div>
      <div>
        <label>Бомбы: </label>
        <input type="number" v-model="bombCount">
      </div>
      <button type="button" @click="restart">Обновить</button>
    </div>
  </div>
</template>

<script>
function getAroundIdx(rowCount, i, j) {
      let indices = [];

      // top-left
      if (i > 0 && j > 0) {
        indices.push([i - 1, j - 1]);
      }
      // top
      if (i > 0) {
        indices.push([i - 1, j]);
      }
      // top-right
      if (i > 0 && j < rowCount - 1) {
        indices.push([i - 1, j + 1]);
      }
      // left
      if (j > 0) {
        indices.push([i, j - 1]);
      }
      // right
      if (j < rowCount - 1) {
        indices.push([i, j + 1]);
      }
      // bottom-left
      if (i < rowCount - 1 && j > 0) {
        indices.push([i + 1, j - 1]);
      }
      // bottom
      if (i < rowCount - 1) {
        indices.push([i + 1, j]);
      }
      // bottom-right
      if (i < rowCount - 1 && j < rowCount - 1) {
        indices.push([i + 1, j + 1]);
      }

      return indices;
    }

    // around === -1 => there is a bomb
    function newBoard(rowCount, bombCount) {
      // init
      let board = [];
      for (let i = 0; i < rowCount; i++) {
        board[i] = [];
        for (let j = 0; j < rowCount; j++) {
          board[i][j] = { around: 0, isMark: false, isClear: false };
        }
      }

      // add bombs
      while (bombCount > 0) {
        let randRow = Math.floor(Math.random() * rowCount);
        let randCol = Math.floor(Math.random() * rowCount);

        if (board[randRow][randCol].around !== -1) {
          board[randRow][randCol].around = -1;
          bombCount--;
        }
      }

      // set around
      for (let i = 0; i < rowCount; i++) {
        for (var j = 0; j < rowCount; j++) {
          if (board[i][j].around === -1) {
            getAroundIdx(rowCount, i, j).forEach(pos => {
              if (board[pos[0]][pos[1]].around !== -1) {
                board[pos[0]][pos[1]].around ++;
              }
            })
          }
        }
      }

      return board;
    }

    function recurClear(row, col, cells, visitedPoses) {
      visitedPoses = visitedPoses || [];

      getAroundIdx(cells.length, row, col).forEach(pos => {
        if (!visitedPoses.some(p => p[0] === pos[0] && p[1] === pos[1])) {
          visitedPoses.push(pos);

          let [x, y] = pos;
          if (cells[x][y].around === 0) {
            cells[x][y].isClear = true;
            recurClear(x, y, cells, visitedPoses);
          } else if (cells[x][y].around > 0) {
            cells[x][y].isClear = true;
          }
        }
      });
    }

    export default{
      data() {
        const rowCount = 4;
        const bombCount = 2;

        return { 
          bombCount,
          rowCount,
          result: '',
          cells: newBoard(rowCount, bombCount) 
        }
      },
      methods: {
        showCell(cell) {
          if (cell.isMark) {
            return '√';
          } else if (cell.isClear) {
            if (cell.around === -1) {
              return 'X';
            } else if (cell.around > 0) {
              return cell.around;
            } else {
              return '';
            }
          } else {
            return '';
          }
        },
        checkWin() {
          let { cells } = this.$data;
          let length = cells.length;
          let unclearBombCount = 0;

          for (let i = 0; i < length; i++) {
            for (let j = 0; j < length; j++) {
              let cell = cells[i][j];

              // not all clear 
              if (cell.around !== -1 && !cell.isClear) {
                return false;
              }

              if (!cell.isClear && cell.around === -1) {
                unclearBombCount++;
              }
            }
          }

          return unclearBombCount == this.bombCount;
        },
        filp(i, j, cell) {
          if (cell.isMark) {
            cell.isMark = false;
            return;
          }

          cell.isClear = true;

          if (cell.around === -1) {
            this.clearAll();
            this.result = 'Game Over';
            return;
          } else {
            // spread around
            if (cell.around === 0) {
              recurClear(i, j, this.cells);
            }

            if (this.checkWin()) {
              this.clearAll();
              this.result = 'You Win';
            }
          } 
        },
        clearAll() {
          let { cells } = this.$data;
          let length = cells.length;

          for (let i = 0; i < length; i++) {
            for (let j = 0; j < length; j++) {
              cells[i][j].isClear = true;
            }
          }
        },
        mark(cell, event) {
          cell.isMark = !cell.isMark;

          event.preventDefault();
        },
        restart() {
          if (this.bombCount >= this.rowCount * this.rowCount) {
            alert('Too many bombs');
            return;
          }
          
          this.cells = newBoard(this.rowCount, this.bombCount);
          this.result = '';
        }
      }
    }
</script>

  <style lang="scss">
  .game-page {
    &__cell {
        width: 50px;
        display: inline-block;
        line-height: 50px;
        height: 50px;
        text-align: center;
        margin: 3px;
        background-color: beige;
        border: 2px solid gray;
        vertical-align: middle;
    }
  }
  .cell 
  {
    width: 50px;
    display: inline-block;
    line-height: 50px;
    height: 50px;
    text-align: center;
    margin: 3px;
    background-color: beige;
    border: 2px solid gray;
    vertical-align: middle;
  }
  .clear 
  {
    background-color: white;
  }
  </style>