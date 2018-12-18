<template>
  <div v-if="socketDataLoaded">
    <div class="game multiplayer">
      <div
        v-for="(cell, index) in board.flatMap(x => x)"
        :key="index"
        :class="`cell-${cell}`"
      />

      <template v-if="pacs">
        <Pac
          v-for="pac in pacs.filter(pac=>pac!==null)"
          :key="`pac-${pac.id}`"
          :class="`pac pac-${pac.id}`"
          :style="pacStyle(pac)"
          :state="pac.state"
          :direction="pac.direction==='none'?pac.initialDirection:pac.direction"
          :hat-color="pac.hatColor"
        />
      </template>

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
        <span> {{pacs.filter(Boolean).length}}/4 Press   <kbd>Space</kbd>
        </span>
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
          :key="`life-${life}`"
        >
          <Pac
            state="eating"
            class="pac"
            direction="right"
            :hat-color="id===null?'red':pacs[id].hatColor"
          />
        </span>
      </div>
      <div>Level: {{ level }}</div>
    </div>

  </div>
</template>


<script>
import Ghost from "./Ghost.vue";
import Pac from "./Pac.vue";
import Vue from "vue";
import VueSocketIO from "vue-socket.io";

Vue.use(
  new VueSocketIO({
    debug: false,
    connection: process.env.VUE_APP_SERVER_ADDRESS
  })
);

const base = 100 / 39;
const unit = "vmin";

export default {
  name: "MultiplayerGame",
  components: {
    Ghost,
    Pac
  },
  data() {
    return {
      id: null,
      countDown: null,
      lifes: null,
      board: null,
      interval: null,
      stopped: null,
      level: null,
      pacs: null,
      blinky: null,
      inky: null,
      pinky: null,
      clyde: null,
      isFirstPlay: null,
      score: null
    };
  },
  computed: {
    socketDataLoaded() {
      return (
        this.countDown !== null &&
        this.lifes !== null &&
        this.board !== null &&
        this.stopped !== null &&
        this.level !== null &&
        this.pacs !== null &&
        this.blinky !== null &&
        this.inky !== null &&
        this.pinky !== null &&
        this.clyde !== null &&
        this.isFirstPlay !== null &&
        this.score !== null
      );
    },
    speed() {
      return Math.max(120 - this.level * 5, 2);
    },
    width() {
      return this.board[0].length;
    },
    height() {
      return this.board.length;
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
    isFirstPlay(value) {
      if (value) {
        window.addEventListener("keydown", this.startFirstPlay);
      } else {
        window.removeEventListener("keydown", this.startFirstPlay);
      }
    },
    stopped(value) {
      if (value) {
        clearInterval(this.interval);
      } else {
        if (this.id !== null) {
          this.interval = setInterval(() => {
            if (this.stopped) {
              clearInterval(this.interval);
              return;
            }
            this.movePac();
          }, this.speed);
        }
      }
    }
  },
  mounted() {
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
  sockets: {
    resetCharacters(data) {
      this.pacs = data;
    },
    id(data) {
      this.id = data;
    },
    countDown(data) {
      this.countDown = data;
    },
    lifes(data) {
      this.lifes = data;
    },
    board(data) {
      this.board = data;
    },
    stopped(data) {
      this.stopped = data;
    },
    level(data) {
      this.level = data;
    },
    pacs(data) {
      const pacs = data;
      if (
        this.id !== null &&
        this.pacs[this.id] !== null &&
        this.pacs[this.id] !== undefined
      ) {
        pacs[this.id] = this.pacs[this.id];
      }
      this.pacs = pacs;
    },
    ghosts(data) {
      const [blinky, pinky, inky, clyde] = data;
      this.blinky = blinky;
      this.pinky = pinky;
      this.inky = inky;
      this.clyde = clyde;
    },
    isFirstPlay(data) {
      this.isFirstPlay = data;
    },
    score(data) {
      this.score = data;
    }
  },
  methods: {
    setNextPacDirection(nextDirection) {
      if (this.id !== null) {
        this.$set(this.pacs[this.id], "nextDirection", nextDirection);
        this.$socket.emit("pac", { ...this.pacs[this.id], nextDirection });
      }
    },
    startFirstPlay(event) {
      const SPACE_KEY = 32;
      if (event.keyCode === SPACE_KEY) {
        this.$socket.emit("stopped", false);
        this.$socket.emit("isFirstPlay", false);
      }
    },
    pacStyle(pac) {
      return {
        top: `${base * pac.y}${unit}`,
        left: `${base * pac.x}${unit}`
      };
    },
    movePac() {
      const pac = this.pacs[this.id];
      if (
        pac.nextDirection !== "none" &&
        this.canGo({ ...pac, direction: pac.nextDirection })
      ) {
        this.$set(pac, "direction", pac.nextDirection);
        this.$set(pac, "nextDirection", "none");
        this.go(pac);
      } else if (this.canGo(pac)) {
        this.go(pac);
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
    }
  }
};
</script>
<style lang="stylus" src="./game.styl">
</style>
