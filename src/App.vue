<template>
  <div id="app">
    <div class="border-1 game">
      <div class="border-2 game-header">
        <div>
          <span
            class="game-header-counter"
          >{{ game.options.bombsCount.toString().padStart(3, '0') }}</span>
        </div>
        <div>
          <div @click="restartGame" class="game-header-emoji" :class="`emoji-${game.status}`"></div>
        </div>
        <div>
          <span class="game-header-counter">{{game.timer.count.toString().padStart(3, '0')}}</span>
        </div>
      </div>
      <div class="border-2 game-campo">
        <ul
          class="game-campo-minas"
          :style="{
            gridTemplateColumns: `repeat(${game.options.size.columns}, 1fr)`,
            width: '260px',
            maxHeight: '260px'
          }"
        >
          <li
            v-for="(_, i) in (game.options.size.rows * game.options.size.columns)"
            :key="i"
            class="game-campo-minas-mina"
            @click="() => { bombClick(i) }"
          >
            <div
              class="mina-overlay"
              :class="{
              closed: !game.discovered.includes(i),
              bomb: game.bombs.includes(i)
            }"
            ></div>

            <div class="bombs-around">{{ getDisplayValue(i) }}</div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      game: {
        status: 'smile',
        timer: {
          running: false,
          count: 0,
          tick: null,
        },
        options: {
          bombsCount: 5,
          size: {
            rows: 6,
            columns: 6,
          },
        },
        bombs: [1, 8, 23, 30, 33],
        discovered: [],
      },
    };
  },
  methods: {
    restartGame() {
      this.game.status = 'smile';
      this.game.discovered = [];
      this.timerAction('reset');
    },
    getTablePosition(i) {
      return {
        row: Math.floor(i / this.game.options.size.rows),
        column: i % this.game.options.size.columns,
      };
    },
    getIndexPosition({ row, column }) {
      return row * this.game.options.size.rows + column;
    },
    getSquaresArround(pos, directContact = true) {
      const firstColumn = pos.column !== 0;
      const lastColumn = pos.column !== this.game.options.size.columns - 1;

      return [
        [pos.row - 1, pos.column - 1, directContact && firstColumn],
        [pos.row - 1, pos.column],
        [pos.row - 1, pos.column + 1, directContact && lastColumn],

        [pos.row, pos.column - 1, firstColumn],
        [pos.row, pos.column + 1, lastColumn],

        [pos.row + 1, pos.column - 1, directContact && firstColumn],
        [pos.row + 1, pos.column],
        [pos.row + 1, pos.column + 1, directContact && lastColumn],
      ]
        .filter(([, , condition = true]) => condition)
        .map(([row, column]) => this.getIndexPosition({ row, column }));
    },
    getBombsArround(pos) {
      return this.getSquaresArround(pos)
        .filter(this.hasBomb)
        .length;
    },
    getSafeSqrsArround(pos) {
      return this.getSquaresArround(pos, true)
        .filter(i => !this.hasBomb(i))
        .filter(i => this.getBombsArround(this.getTablePosition(i)) === 0);
    },
    getDisplayValue(i) {
      const pos = this.getTablePosition(i);
      if (this.getBombsArround(pos) > 0 && this.game.discovered.includes(i)) {
        return this.getBombsArround(pos);
      }

      return '';
    },
    hasBomb(i) {
      if (i >= 0 && i < this.game.options.size.rows * this.game.options.size.columns) {
        return this.game.bombs.includes(i);
      }
      return false;
    },
    bombClick(i) {
      if (this.game.status === 'xx') return;

      this.discover(i);
      if (this.game.bombs.includes(i)) {
        this.timerAction('clearInterval');
        this.game.bombs.forEach(this.discover);
        this.game.status = 'xx';
      } else if (this.game.timer.running === false) {
        this.timerAction('start');
      }
    },

    discover(i) {
      if (!this.game.discovered.includes(i)) {
        this.game.discovered.push(i);
      }
    },

    timerAction(action) {
      const getTickInterval = () => setInterval(() => { this.game.timer.count += 1; }, 1000);

      const actions = {
        start: () => {
          this.game.timer.tick = getTickInterval();
          this.game.timer.running = true;
        },
        clearInterval: () => {
          clearInterval(this.game.timer.tick);
          this.game.timer.running = false;
        },
        clearCount: () => { this.game.timer.count = 0; },

        pause: () => {
          if (this.game.timer.running) {
            actions.clearInterval();
          } else {
            actions.start();
          }
        },

        reset: () => { actions.clearInterval(); actions.clearCount(); },
      };

      actions[action]();
    },
  },
};
</script>

<style lang="scss">
@import "~@/assets/scss/app";

$border-width: 4px;
$border-light: $border-width solid rgba(#ededfc, 0.4);
$border-shadow: $border-width solid rgba(#030220, 0.4);

.border-1 {
  border-top: $border-light;
  border-left: $border-light;

  border-bottom: $border-shadow;
  border-right: $border-shadow;
}

.border-2 {
  border-top: $border-shadow;
  border-left: $border-shadow;

  border-bottom: $border-light;
  border-right: $border-light;
}

#app,
.game {
  height: 100%;
  min-height: 100vh;
}

.game {
  width: 100%;
  height: 100%;

  background-color: #c4c4c4;
  padding: 4px;

  display: flex;
  flex-direction: column;

  .game-header {
    display: flex;
    justify-content: space-between;
    align-items: center;

    padding: 6px;

    &-counter {
      display: block;

      background-color: #3b0908;
      border-radius: 4px;

      padding: 6px;

      font-size: 18px;
      letter-spacing: 0.125em;
      color: #fa2622;
    }

    &-emoji {
      width: 32px;
      height: 32px;

      $emojis: "smile", "surprise", "xx", "brbo";

      @each $emoji in $emojis {
        &.emoji-#{$emoji} {
          background: url("~@/assets/icons/emoji-#{$emoji}.png");
          background-size: 100%;
        }
      }
    }
  }

  .game-campo {
    flex: 1;
    padding: 4px;

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    &-minas {
      border: 2px solid rgba(#030220, 0.2);
      flex: 1;

      display: grid;

      &-mina {
        $bg-mine: rgba(8, 18, 19, 0.31);
        display: flex;
        justify-content: center;
        align-items: center;
        position: relative;
        width: 100%;

        background-color: white;

        .mina-overlay {
          position: absolute;
          top: 0;
          left: 0;

          width: 100%;
          height: 100%;

          background: $bg-mine;

          &.bomb {
            background: url("~@/assets/icons/emoji-bomb.png") $bg-mine center
              center no-repeat;

            background-size: 70%;
          }
          &.closed {
            border-top: $border-light;
            border-left: $border-light;

            border-bottom: $border-shadow;
            border-right: $border-shadow;

            background: linear-gradient(
                0deg,
                rgba(8, 18, 19, 0.31),
                rgba(8, 18, 19, 0.31)
              ),
              #ffffff;
          }
        }

        .bombs-around {
          max-width: 75%;
          max-height: 75%;
          width: 1rem;
          height: 1rem;
          text-align: center;
          user-select: none;
          font-size: 1rem;
        }
      }
    }
  }
}
</style>
