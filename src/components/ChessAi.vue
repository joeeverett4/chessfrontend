<template>
  <div>
    <div
      class="d-flex"
      v-for="(outerArray, outerIndex) in nestedArray"
      :key="outerIndex"
    >
      <!-- WHite player -->
      <!-- Black player -->
      <p>{{ names[outerIndex]?.white }} vs {{ names[outerIndex]?.black }}</p>
    
      <div class = "chess-container">
      <chessboard :fen="positions[outerIndex]" />

      <div class="controls">
          <!-- Previous button -->
      <svg
        @click="goToMove(-1, outerIndex)"
        :class="{ disabled: count === 0 }"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <polyline points="15 18 9 12 15 6"></polyline>
      </svg>
      
      <!-- Next button -->
      <svg
        @click="goToMove(1, outerIndex)"
        :class="{ disabled: count === pgns[outerIndex].split(' ').length - 1 }"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <polyline points="9 18 15 12 9 6"></polyline>
      </svg>
      </div>
     <div class = "move-container"> 
     <div class = "move-list"> 
      <div style = "margin-bottom:10px;" v-for="i in outerArray[0]" :key="i" >
         {{i}} 
        </div>  
     </div>  
      <div class="move-grid">
        <div v-for="(innerObject, innerIndex) in outerArray.slice(1)" :key="innerIndex">
          <p
            :style="getParagraphStyle(outerArray[innerIndex + 1]?.next_mistake)"
            @click="goToMoveOnBoard(outerIndex, innerIndex)"
          >
            {{ innerObject.move }}
          </p>
        </div>
      </div>
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
      names: [],
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

    goToMoveOnBoard(chessboardIndex, moveIndex) {
      // Update the count and position to reflect the selected move
      this.count = moveIndex;
      this.positions[chessboardIndex] = this.getPositionForMove(
        this.count,
        chessboardIndex
      );
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
          this.names = data.objects.map((game) => ({
            black: game.players.black.user.name,
            white: game.players.white.user.name,
          }));

          console.log(this.names);

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
  margin-left:10px;
  
}

.move-container{
    display:flex;
    max-height: 400px;
    overflow: scroll;
    padding-right: 20px;

}
.move-list{
 margin-left:20px;
}
.d-flex {
  margin: 0 auto;
  max-width: 1000px;
  padding-bottom: 50px;
  position:relative;
}
.d-flex p{
    text-align: left;
}
.chess-container{
  display: flex;
  margin: 0 auto;
  max-width: 1000px;
  position: relative;
}
.controls{
    display: flex;
    position:absolute;
    bottom:6%;
    left:8%;
}
.controls svg{
    width:40px;
}
p {
  margin-block-start: 0;
  margin-block-end: 0;
}
.disabled {
  opacity: 0.5; /* Add a bit of transparency for disabled buttons */
  cursor: not-allowed; /* Change cursor to indicate non-interactivity */
}

</style>
