<script lang="ts">
  import { onMount } from "svelte";
  import "chessboard-element";
  import { Chess } from "chess.js";
  let chessboard;

  onMount(() => {
    const chess = new Chess();
    const stockfish = new Worker("stockfish.js");
    stockfish.postMessage("uci");
    stockfish.postMessage("ucinewgame");
    stockfish.postMessage("position startpos");

    chessboard.setPosition("start");
    chessboard.draggablePieces = true;

    chessboard.addEventListener("drop", (event) => {
      try {
        chess.move({
          from: event.detail.source,
          to: event.detail.target,
        });
      } catch (e) {
        // TODO: reverting chess board to previous position not working
        chessboard.setPosition(chess.fen());
      }

      stockfish.postMessage("position fen " + chess.fen());
      stockfish.postMessage("go depth 10");
    });

    stockfish.onmessage = (event) => {
      if (event.data.includes("bestmove")) {
        const bestMove = event.data.split(" ")[1];
        const move = chess.move(bestMove);
        console.log(move);
        chessboard.setPosition(chess.fen());
      }
      console.log(event.data);
    };
  });
</script>

<main>
  <h1>Vite + Svelte</h1>
  <chess-board bind:this={chessboard} style="width: 400px" id="board" />
</main>

<style>
</style>
