<div class="cube">
  <div class="side front" />
  <div class="side back" />
  <div class="side left" />
  <div class="side right" />
  <div class="side top" />
  <div class="side bottom" />
</div>

<style>
  .cube {
    width: 100px;
    height: 100px;
    position: relative;
    transform-style: preserve-3d;
    animation: spin 5s linear infinite;
  }

  .side {
    position: absolute;
    width: 100px;
    height: 100px;
  }

  .front {
    transform: translateZ(50px);
    background-color: red;
  }

  .back {
    transform: rotateY(180deg) translateZ(50px);
    background-color: blue;
  }

  .left {
    transform: rotateY(-90deg) translateZ(50px);
    background-color: green;
  }

  .right {
    transform: rotateY(90deg) translateZ(50px);
    background-color: yellow;
  }

  .top {
    transform: rotateX(90deg) translateZ(50px);
    background-color: purple;
  }

  .bottom {
    transform: rotateX(-90deg) translateZ(50px);
    background-color: orange;
  }

  @keyframes spin {
    0% {
      transform: rotateY(0deg);
    }
    100% {
      transform: rotateY(360deg);
    }
  }
</style>
