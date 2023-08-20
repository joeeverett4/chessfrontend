<template>
  <div>
      <div>
    <div class = "d-flex" v-for="(outerArray, outerIndex) in nestedArray" :key="outerIndex">
        <chessboard :fen="position" />
      <div class="controls">
      <button @click="goToMove(-1)" :disabled="count === 0">Previous</button>
      <button @click="goToMove(1)" :disabled="count === pgnMoves.length - 1">Next</button>
    </div>
      <div class = "move-grid">  
      <div v-for="(innerObject, innerIndex) in outerArray" :key="innerIndex">
       <p :style="getParagraphStyle(innerObject.mistake)"> {{ innerObject.move}} </p> <!-- Output the value of the key you want -->
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
      position: "start",
      count: 0,
      pgnMoves: [
        "e4",
        "e5",
        "Nf3",
        "Nc6",
        "Bc4",
        "Bc5",
        "d3",
        "d6",
        "h3",
        "Nf6",
        "Bg5",
        "O-O",
        "Nbd2",
        "Be6",
        "c3",
        "Qe7",
      ],
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
  
     goToMove(step) {
         console.log("goToMove")
         console.log(this.position)
      if (this.count + step >= 0 && this.count + step < this.pgnMoves.length) {
        this.count += step;
        this.position = this.getPositionForMove(this.count);
      }
    },
    getPositionForMove(moveIndex) {
      console.log("getPositionForMove")
      console.log(moveIndex)  
      let position = new Chess();
      for (let i = 0; i <= moveIndex; i++) {
        position.move(this.pgnMoves[i]);
      }
      return position.fen();
    },
   
    fetchGames() {
      console.log("fetch games");
      fetch("http://localhost:5000/get_games") // Make GET request to Flask server
        .then((response) => response.json()) // Parse the response as JSON
        .then((data) => {
          console.log(data); // Display the fetched games data in the console
          this.nestedArray = data
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
         // console.log(data.objects[0].moves); // Display the fetched games data in the console
          let arr = data.objects[0].moves.split(" ")
          this.pgnMoves = arr
          this.pgns = data.objects
          console.log(this.pgns)
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
.move-grid{
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* Create two equally sized columns */
  gap: 10px; /* Add some space between the moves */
}
.d-flex{
  display: flex;
  margin: 0 auto;
  max-width:1000px;  
}
</style>
