<template>
  <div>
    <div class="game">
      <div
        v-for="(cell, index) in board.flatMap(x => x)"
        :key="index"
        :class="`cell-${cell}`"
      />
      <Pac
        class="pac"
        :style="pacStyle"
        :state="pac.state"
        :direction="pac.direction"
        hat-color="red"
      />
      <Ghost
        class="blinky"
        :style="blinkyStyle"
      />
      <Ghost
        class="pinky"
        :style="pinkyStyle"
      />
      <Ghost
        :style="inkyStyle"
        class="inky"
      />
      <Ghost
        class="clyde"
        :style="clydeStyle"
      />

      <div
        v-show="countDown"
        class="restart-message"
      >
        <span>Starting in</span>
        <span>{{ countDown }}</span>
      </div>
      <div
        v-show="isFirstPlay"
        class="start-message"
      >
        <span>Press <kbd>Space</kbd> </span>
        <span>to start</span>
      </div>

      <img
        class="church-1"
        src="@/assets/background-church-1.png"
      >
      <img
        class="church-2"
        src="@/assets/background-church-2.png"
      >
      <img
        class="whatever-1"
        src="@/assets/background-whatever-1.svg"
      >
      <img
        class="tower"
        src="@/assets/background-tower.png"
      >
      <img
        class="building-2"
        src="@/assets/background-building-2.png"
      >
      <img
        class="building-3"
        src="@/assets/background-building-3.svg"
      >
      <img
        class="building-4"
        src="@/assets/background-building-4.svg"
      >
      <img
        class="building-6"
        src="@/assets/background-building-6.png"
      >
      <img
        class="building-1"
        src="@/assets/background-building-1.png"
      >
      <img
        class="balls"
        src="@/assets/background-balls.png"
      >
      <img
        class="building-5"
        src="@/assets/background-building-5.png"
      >
    </div>

    <div class="stats">
      <div> Score: {{ score }}</div>
      <div>
        <span
          v-for="life in lifes"
          :key="life"
        >
          <Pac
            state="eating"
            class="pac"
            direction="right"
            hat-color="red"
          />
        </span>
      </div>
      <div>Level: {{ level }}</div>
    </div>
  </div>
</template>

<script>
import { board } from "./board.json";
import Ghost from "./Ghost.vue";
import Pac from "./Pac.vue";
import shuffle from "fisher-yates/inplace";

const base = 100 / 39;
const unit = "vmin";
const items = [2, 3, 4, 5];

const pacInitial = () => ({
  x: 0,
  y: 6,
  direction: "none",
  nextDirection: "none",
  state: "normal"
});

const blinkyInitial = () => ({
  x: 26,
  y: 17,
  direction: "right"
});

const pinkyInitial = () => ({
  x: 23,
  y: 17,
  direction: "down"
});

const inkyInitial = () => ({
  x: 17,
  y: 15,
  direction: "right"
});
const clydeInitial = () => ({
  x: 11,
  y: 11,
  direction: "right"
});

export default {
  name: "Game",
  components: {
    Ghost,
    Pac
  },
  data() {
    return {
      countDown: 0,
      lifes: 3,
      board,
      interval: null,
      stopped: true,
      level: 1,
      pac: pacInitial(),
      blinky: blinkyInitial(),
      inky: inkyInitial(),
      pinky: pinkyInitial(),
      clyde: clydeInitial(),
      isFirstPlay: true,
      score: 0
    };
  },
  computed: {
    speed() {
      return Math.max(120 - this.level * 5, 2);
    },
    width() {
      return this.board[0].length;
    },
    height() {
      return this.board.length;
    },
    pacStyle() {
      return {
        top: `${base * this.pac.y}${unit}`,
        left: `${base * this.pac.x}${unit}`
      };
    },
    blinkyStyle() {
      return {
        top: `${base * this.blinky.y}${unit}`,
        left: `${base * this.blinky.x}${unit}`
      };
    },
    pinkyStyle() {
      return {
        top: `${base * this.pinky.y}${unit}`,
        left: `${base * this.pinky.x}${unit}`
      };
    },
    inkyStyle() {
      return {
        top: `${base * this.inky.y}${unit}`,
        left: `${base * this.inky.x}${unit}`
      };
    },
    clydeStyle() {
      return {
        top: `${base * this.clyde.y}${unit}`,
        left: `${base * this.clyde.x}${unit}`
      };
    }
  },
  watch: {
    isFirstPlay: {
      handler(value) {
        if (value) {
          window.addEventListener("keydown", this.startFirstPlay);
        } else {
          window.removeEventListener("keydown", this.startFirstPlay);
        }
      },
      immediate: true
    },
    stopped(value) {
      if (value) {
        clearInterval(this.interval);
      } else {
        this.interval = setInterval(() => {
          if (this.stopped) {
            clearInterval(this.interval);
            return;
          }
          this.movePac();
          this.checkItems();
          this.checkCollisions();
          this.moveBlinky();
          this.movePinky();
          this.moveInky();
          this.moveClyde();
          this.checkCollisions();
        }, this.speed);
      }
    }
  },
  mounted() {
    this.placeItems();
    window.addEventListener("keydown", event => {
      const ARROW_LEFT = 37;
      const ARROW_UP = 38;
      const ARROW_RIGHT = 39;
      const ARROW_DOWN = 40;
      switch (event.keyCode) {
        case ARROW_LEFT:
          this.setNextPacDirection("left");
          break;
        case ARROW_RIGHT:
          this.setNextPacDirection("right");
          break;
        case ARROW_UP:
          this.setNextPacDirection("up");
          break;
        case ARROW_DOWN:
          this.setNextPacDirection("down");
          break;
        default:
          break;
      }
    });
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
  methods: {
    startFirstPlay(event) {
      const SPACE_KEY = 32;
      if (event.keyCode === SPACE_KEY) {
        this.stopped = false;
        this.isFirstPlay = false;
      }
    },
    canGo({ x, y, direction }) {
      switch (direction) {
        case "left":
          return this.canGoLeft({ x, y });
        case "right":
          return this.canGoRight({ x, y });
        case "up":
          return this.canGoUp({ x, y });
        case "down":
          return this.canGoDown({ x, y });
        case "none":
          return true;
        default:
          throw new Error(`unknown direction ${direction}`);
      }
    },
    canGoRight({ x, y }) {
      return this.board[y][(x + 1) % this.width] !== 1;
    },
    canGoLeft({ x, y }) {
      return this.board[y][(x - 1 + this.width) % this.width] !== 1;
    },
    canGoUp({ x, y }) {
      return this.board[(y - 1 + this.height) % this.height][x] !== 1;
    },
    canGoDown({ x, y }) {
      return this.board[(y + 1) % this.height][x] !== 1;
    },
    go(obj) {
      switch (obj.direction) {
        case "left":
          return this.goLeft(obj);
        case "right":
          return this.goRight(obj);
        case "up":
          return this.goUp(obj);
        case "down":
          return this.goDown(obj);
        case "none":
          break;
        default:
          throw new Error("unknown direction");
      }
    },
    goRight(object) {
      this.$set(object, "x", (object.x + 1) % this.width);
    },
    goLeft(object) {
      this.$set(object, "x", (object.x - 1 + this.width) % this.width);
    },
    goUp(object) {
      this.$set(object, "y", (object.y - 1 + this.height) % this.height);
    },
    goDown(object) {
      this.$set(object, "y", (object.y + 1) % this.height);
    },
    moveRandom(obj) {
      let possibleDirections = [];
      if (this.canGoLeft(obj)) {
        possibleDirections.push("left");
      }
      if (this.canGoRight(obj)) {
        possibleDirections.push("right");
      }
      if (this.canGoDown(obj)) {
        possibleDirections.push("down");
      }
      if (this.canGoUp(obj)) {
        possibleDirections.push("up");
      }
      const setRandomDirection = obj => {
        const randomDirectionIndex = Math.floor(
          Math.random() * possibleDirections.length
        );
        const randomDirection = possibleDirections[randomDirectionIndex];
        this.$set(obj, "direction", randomDirection);
      };
      if (this.canGo(obj)) {
        const opposites = {
          left: "right",
          right: "left",
          up: "down",
          down: "up"
        };
        const notReverseDirections = possibleDirections.filter(direction => {
          return direction !== opposites[obj.direction];
        });
        if (notReverseDirections.length > 0) {
          const randomNotReverseDirectionIndex = Math.floor(
            Math.random() * notReverseDirections.length
          );
          const randomNotReverseDirection =
            notReverseDirections[randomNotReverseDirectionIndex];
          if (Math.random() > 0.01) {
            this.$set(obj, "direction", randomNotReverseDirection);
          } else {
            setRandomDirection(obj);
          }
        } else {
          setRandomDirection(obj);
        }
      } else {
        setRandomDirection(obj);
      }
      this.go(obj);
    },
    movePac() {
      if (
        this.pac.nextDirection !== "none" &&
        this.canGo({ ...this.pac, direction: this.pac.nextDirection })
      ) {
        this.$set(this.pac, "direction", this.pac.nextDirection);
        this.$set(this.pac, "nextDirection", "none");
        this.go(this.pac);
      } else if (this.canGo(this.pac)) {
        this.go(this.pac);
      }
    },
    checkItems() {
      if (items.includes(this.board[this.pac.y][this.pac.x])) {
        this.$set(this.board[this.pac.y], this.pac.x, 0);
        this.score += 10;
        this.$set(this.pac, "state", "eating");
        if (this.score % 50 === 0) {
          this.level++;
          this.countDown = 5;
          this.stopped = true;
          this.resetCharacters();
          this.placeItems();
          const interval = setInterval(() => {
            this.countDown--;
            if (this.countDown === 0) {
              this.stopped = false;
              clearInterval(interval);
            }
          }, 1000);
        }
      } else {
        this.$set(this.pac, "state", "normal");
      }
    },
    checkCollisions() {
      const hasLost = [this.blinky, this.pinky, this.inky, this.clyde].some(
        ghost => ghost.x === this.pac.x && ghost.y === this.pac.y
      );
      if (hasLost) {
        if (this.lifes > 1) {
          this.lifes--;
          this.countDown = 5;
          this.stopped = true;
          const interval = setInterval(() => {
            this.countDown--;
            if (this.countDown === 0) {
              this.resetCharacters();
              this.stopped = false;
              clearInterval(interval);
            }
          }, 1000);
        } else {
          if (!this.stopped) {
            this.stopped = true;
            setTimeout(() => {
              this.placeItems();
              this.resetCharacters();
              this.score = 0;
              this.lifes = 3;
              this.level = 1;
              this.isFirstPlay = true;
            }, 3000);
          }
        }
      }
    },
    moveBlinky() {
      this.moveRandom(this.blinky);
    },
    moveClyde() {
      this.moveRandom(this.clyde);
    },
    movePinky() {
      this.moveRandom(this.pinky);
    },
    moveInky() {
      this.moveRandom(this.inky);
    },
    freePositions(n) {
      const freePositions = this.board
        .flatMap((row, y) => row.map((cell, x) => ({ x, y })))
        .filter(({ x, y }) => this.board[y][x] === 0)
        .filter(({ x, y }) => x !== this.pac.x && y !== this.pac.y);
      const firstN = shuffle(freePositions).slice(0, n);
      return firstN;
    },
    resetCharacters() {
      this.$set(this, "pac", pacInitial());
      this.$set(this, "blinky", blinkyInitial());
      this.$set(this, "pinky", pinkyInitial());
      this.$set(this, "inky", inkyInitial());
      this.$set(this, "clyde", clydeInitial());
    },
    placeItems() {
      const firstFive = this.freePositions(5);
      for (const { x, y } of firstFive) {
        const randomItemIndex = Math.floor(Math.random() * items.length);
        const randomItem = items[randomItemIndex];
        this.$set(this.board[y], x, randomItem);
      }
    },
    setPacDirection(direction) {
      this.$set(this.pac, "direction", direction);
    },
    setNextPacDirection(direction) {
      this.$set(this.pac, "nextDirection", direction);
    }
  }
};
</script>

<style lang="stylus" src="./game.styl">
</style>
