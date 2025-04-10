<script>
  import Game from './lib/Game.svelte'
</script>

<main>
  <h1>벽돌깨기 게임</h1>
  <Game />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 800px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 2em;
    font-weight: 100;
    margin-bottom: 1em;
  }
</style>
