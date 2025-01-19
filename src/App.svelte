<script>
  // @ts-nocheck
  import { onMount, tick } from 'svelte';
  
  let carousel;
  let angle = 0;
  let selectedIndex = 0;
  let isDragging = false;
  let startX = 0;
  let currentX = 0;
  let autoRotate = false;
  let autoRotateInterval;
  
  const cellSize = 210;
  const minCellCount = 3;
  const maxCellCount = 50;
  let cellCount = 9;
  let radius;
  
  function calculateRadius() {
    radius = Math.round((cellSize / 2) / Math.tan(Math.PI / cellCount));
  }
  
  async function updateCarouselCells() {
    await tick();
    const cells = carousel.children;
    for (let i = 0; i < cells.length; i++) {
      const cellAngle = (i / cellCount) * 360;
      cells[i].style.transform = `rotateY(${cellAngle}deg) translateZ(${radius}px)`;
      cells[i].style.display = i < cellCount ? 'block' : 'none';
      cells[i].classList.remove('selected');
    }
    cells[selectedIndex]?.classList.add('selected');
  }
  
  function rotateCarousel(direction = 1) {
    const rotationAngle = 360 / cellCount;
    angle += direction * rotationAngle;
    carousel.style.transform = `translateZ(-${radius}px) rotateY(${angle}deg)`;
  }
  
  function handleCellClick(index) {
    const cells = carousel.children;
    selectedIndex = index;
    const selectedAngle = -index * (360 / cellCount);
    angle = selectedAngle;
    carousel.style.transform = `translateZ(-${radius}px) rotateY(${selectedAngle}deg)`;
    for (let i = 0; i < cells.length; i++) {
      cells[i].classList.remove('selected');
    }
    cells[index].classList.add('selected');
  }
  
  function handleKeyDown(event, index) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      handleCellClick(index);
    }
  }
  
  function handleMouseDown(event) {
    isDragging = true;
    startX = event.clientX;
  }
  //
  function handleMouseMove(event) {
    if (isDragging) {
      currentX = event.clientX;
      const deltaX = currentX - startX;
      const rotationAngle = (deltaX / cellSize) * (360 / cellCount);
      angle -= rotationAngle;
      carousel.style.transform = `translateZ(-${radius}px) rotateY(${angle}deg)`;
      startX = currentX;
    }
  }
  
  function handleMouseUp() {
    isDragging = false;
  }
  
  function toggleAutoRotate() {
    autoRotate = !autoRotate;
    if (autoRotate) {
      autoRotateInterval = setInterval(() => rotateCarousel(1), 2000);
    } else {
      clearInterval(autoRotateInterval);
    }
  }
  
  function handleSliderChange(event) {
    cellCount = event.target.value;
    calculateRadius();
    updateCarouselCells();
    angle = 0;
    selectedIndex = 0;
    carousel.style.transform = `translateZ(-${radius}px)`;
  }
  
  onMount(() => {
    calculateRadius();
    updateCarouselCells();
  });
</script>

<style>
  .scene {
    
    margin: 40px 0;
    position: relative;
    width: 210px;
    height: 140px;
    margin: 80px auto;
    perspective: 1000px;
    border-radius: 25px;
  }

  .carousel {
    width: 100%;
    height: 100%;
    position: absolute;
    transform: translateZ(-288px);
    transform-style: preserve-3d;
    transition: transform 1s;
  }

  .carousel__cell {
    position: absolute;
    width: 190px;
    height: 120px;
    left: 10px;
    top: 10px;
    border: 2px solid black;
    line-height: 116px;
    font-size: 80px;
    font-weight: bold;
    color: white;
    text-align: center;
    transition: transform 1s, opacity 1s;
    cursor: pointer;
    transform-origin: center;
    border-radius: 25px;
  }

  .carousel__cell:focus {
    outline: 3px solid #007bff;
    outline-offset: 2px;
    width: 200px;
    height: 130px;
  }

  

  .controls {
    text-align: center;
    margin: 20px 0;
  }

  button {
    margin: 5px;
    padding: 8px 16px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:hover {
    background: #0056b3;
  }

  button:focus {
    outline: 3px solid #0056b3;
    outline-offset: 2px;
  }

  .carousel__cell:nth-child(9n+1) { background: hsla(  0, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+2) { background: hsla( 40, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+3) { background: hsla( 80, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+4) { background: hsla(120, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+5) { background: hsla(160, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+6) { background: hsla(200, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+7) { background: hsla(240, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+8) { background: hsla(280, 100%, 50%, 0.8); }
  .carousel__cell:nth-child(9n+0) { background: hsla(320, 100%, 50%, 0.8); }

  @media (max-width: 768px) {
    .scene {
      width: 180px;
      height: 120px;
    }

    .carousel__cell {
      width: 160px;
      height: 100px;
      font-size: 60px;
      line-height: 96px;
    }
  }
</style>

<div 
  class="scene"
  role="region"
  aria-label="3D Carousel">
  <div class="carousel" bind:this={carousel}>
    {#each Array(maxCellCount) as _, i}
      <button
        class="carousel__cell"
        aria-label="Carousel item {i + 1}"
        onclick={() => handleCellClick(i)}
        onkeydown={(e) => handleKeyDown(e, i)}>
        {i + 1}
      </button>
    {/each}
  </div>
</div>


<div class="controls">
  <button onclick={() => rotateCarousel(-1)}>Rotate Left</button>
  <button onclick={() => rotateCarousel(1)}>Rotate Right</button>
  <button onclick={toggleAutoRotate}>
    {autoRotate ? 'Stop Auto Rotate' : 'Start Auto Rotate'}
  </button>
  <label for="cell-slider">Number of Cells:</label>
  <input
    id="cell-slider"
    type="range"
    min={minCellCount}
    max={maxCellCount}
    value={cellCount}
    oninput={handleSliderChange}
  />
</div>