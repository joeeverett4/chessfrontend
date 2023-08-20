<template>
  <div>
    <div
      class="d-flex"
      v-for="(outerArray, outerIndex) in nestedArray"
      :key="outerIndex"
    >
      <chessboard :fen="positions[outerIndex]" />
      <div class="controls">
        <button @click="goToMove(-1, outerIndex)" :disabled="count === 0">
          Previous
        </button>
        <button
          @click="goToMove(1, outerIndex)"
          :disabled="count === pgns[outerIndex].split(' ').length - 1"
        >
          Next
        </button>
      </div>
      <div class="move-grid">
        <div v-for="(innerObject, innerIndex) in outerArray" :key="innerIndex">
          <p :style="getParagraphStyle(outerArray[innerIndex + 1]?.next_mistake)">
            {{ innerObject.move }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { chessboard } from "vue-chessboard";
import "vue-chessboard/dist/vue-chessboard.css";
import { Chess } from "chess.js";

export default {
  components: {
    chessboard,
  },

  data() {
    return {
      nestedArray: [], // This will hold your API data
      pgns: [],
      positions: [],
      count: 0,
    };
  },
  mounted() {
    this.fetchJson();
    this.fetchGames();
  },
  methods: {
    increment() {
      this.count++;
    },
    getParagraphStyle(isMistake) {
      if (isMistake === "true") {
        return { color: "red" }; // Set the color to red for mistakes
      } else {
        return { color: "black" }; // Set the color to black for non-mistakes
      }
    },

    goToMove(step, chessboardIndex) {
      if (
        this.count + step >= 0 &&
        this.count + step < this.pgns[chessboardIndex].split(" ").length
      ) {
        this.count += step;
        this.positions[chessboardIndex] = this.getPositionForMove(
          this.count,
          chessboardIndex
        );
      }
    },

    getPositionForMove(moveIndex, chessboardIndex) {
      let position = new Chess();
      let moves = this.pgns[chessboardIndex].split(" ");
      for (let i = 0; i <= moveIndex; i++) {
        position.move(moves[i]);
      }
      return position.fen();
    },

    fetchGames() {
      console.log("fetch games");
      fetch("http://localhost:5000/get_games") // Make GET request to Flask server
        .then((response) => response.json()) // Parse the response as JSON
        .then((data) => {
          console.log(data); // Display the fetched games data in the console
          this.nestedArray = data;
        })
        .catch((error) => {
          console.error("Error fetching games:", error);
        });
    },

    fetchJson() {
      console.log("fetch games");
      fetch("http://localhost:5000/get_pgn") // Make GET request to Flask server
        .then((response) => response.json()) // Parse the response as JSON
        .then((data) => {
          console.log(data.objects); // Display the fetched games data in the console
          // Initialize the pgnArray with individual PGNs
          this.pgns = data.objects.map((game) => game.moves);
         // console.log(this.pgns);
          this.positions = new Array(data.length).fill("start");
        })
        .catch((error) => {
          console.error("Error fetching games:", error);
        });
    },
  },
};
</script>

<style>
/* You can add some basic styling here */
.move-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* Create two equally sized columns */
  gap: 10px; /* Add some space between the moves */
  max-height: 400px;
  overflow: scroll;
  padding-right:35px;
}
.d-flex {
  display: flex;
  margin: 0 auto;
  max-width: 1000px;
  padding-bottom:50px;
}
p{
    margin-block-start: 0;
    margin-block-end:0;
}
</style>
