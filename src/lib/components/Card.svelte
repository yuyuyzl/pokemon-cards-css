<script>

  /**
 * return a value that has been rounded to a set precision
 * @param {Number} value the value to round
 * @param {Number} precision the precision (decimal places), default: 3
 * @returns {Number}
 */
const round = (value, precision = 3) => parseFloat(value.toFixed(precision));

/**
 * return a value that has been limited between min & max
 * @param {Number} value the value to clamp
 * @param {Number} min minimum value to allow, default: 0
 * @param {Number} max maximum value to allow, default: 100
 * @returns {Number}
 */
const clamp = (value, min = 0, max = 100 ) => {
	return Math.min(Math.max(value, min), max);
};

/**
 * return a value that has been re-mapped according to the from/to
 * - for example, adjust(10, 0, 100, 100, 0) = 90
 * @param {Number} value the value to re-map (or adjust)
 * @param {Number} fromMin min value to re-map from
 * @param {Number} fromMax max value to re-map from
 * @param {Number} toMin min value to re-map to
 * @param {Number} toMax max value to re-map to
 * @returns {Number} 
 */
const adjust = (value, fromMin, fromMax, toMin, toMax) => {
	return round(toMin + (toMax - toMin) * (value - fromMin) / (fromMax - fromMin));
};
debugger;
  // data / pokemon props
  export let id = "";
  export let name = "";
  export let number = "";
  export let set = "";
  export let types = "";
  export let subtypes = "basic";
  export let supertype = "pokémon";
  export let rarity = "common";

  // image props
  export let img = "";
  export let back = "https://tcg.pokemon.com/assets/img/global/tcg-card-back-2x.jpg";
  export let foil = "";
  export let mask = "";

  // context/environment props
  export let showcase = false;

  const randomSeed = {
    x: Math.random(),
    y: Math.random()
  }

  const cosmosPosition = { 
    x: Math.floor( randomSeed.x * 734 ), 
    y: Math.floor( randomSeed.y * 1280 ) 
  };

  let isTrainerGallery = false;

  let back_img = back;
  let img_base = img.startsWith("http") ? "" : "https://images.pokemontcg.io/";


  let thisCard;

  let active = false;
  let interacting = false;
  let loading = false;
  let isVisible = document.visibilityState === "visible";

  // 仅存储基本数据
  let degrees = { x: 0, y: 0 };
  let opacity = 0;
  let rotateDelta = { x: 0, y: 0 };
  let translateValues = { x: 0, y: 0 };
  let scaleValue = 1;


  const interact = (e) => {
    

    if (!isVisible) {
      return (interacting = false);
    }

    interacting = true;

    if (e.type === "touchmove") {
      e.clientX = e.touches[0].clientX;
      e.clientY = e.touches[0].clientY;
    }

    const $el = e.target;
    const rect = $el.getBoundingClientRect(); // get element's current size/position
    const absolute = {
      x: e.clientX - rect.left, // get mouse position from left
      y: e.clientY - rect.top, // get mouse position from right
    };
    const percent = {
      x: clamp(round((100 / rect.width) * absolute.x)),
      y: clamp(round((100 / rect.height) * absolute.y)),
    };
    const center = {
      x: percent.x - 50,
      y: percent.y - 50,
    };

    // 更新基本值，只保存中心距离
    degrees = {
      x: round(-(center.x / 3.5)),
      y: round(center.y / 2)
    };
    opacity = 1;
  };

  const interactEnd = (e, delay = 500) => {
    setTimeout(function () {
      interacting = false;
      degrees = { x: 0, y: 0 };
      opacity = 0;
    }, delay);
  };



  let foilStyles = `
    --mask: url(${mask});
    --foil: url(${foil});`;
  const staticStyles = `
    --seedx: ${randomSeed.x};
    --seedy: ${randomSeed.y};
    --cosmosbg: ${cosmosPosition.x}px ${cosmosPosition.y}px;
  `;
  $: dynamicStyles = `
    --pointer-x: ${50 - degrees.x * 1.5}%;
    --pointer-y: ${50 + degrees.y * 1.5}%;
    --pointer-from-center: ${ 
      clamp( Math.sqrt( 
        degrees.y * degrees.y + 
        degrees.x * degrees.x
      ) / 10, 0, 1) };
    --pointer-from-top: ${(50 + degrees.y * 1.5) / 100};
    --pointer-from-left: ${(50 - degrees.x * 1.5) / 100};
    --card-opacity: ${opacity};
    --rotate-x: ${degrees.x + rotateDelta.x}deg;
    --rotate-y: ${degrees.y + rotateDelta.y}deg;
    --background-x: ${adjust(-degrees.x, -30, 30, 37, 63)}%;
    --background-y: ${adjust(degrees.y, -30, 30, 33, 67)}%;
    --card-scale: ${scaleValue};
    --translate-x: ${translateValues.x}px;
    --translate-y: ${translateValues.y}px;
	`;

  $: {
    rarity = rarity.toLowerCase();
    supertype = supertype.toLowerCase();
    number = number.toLowerCase();
    isTrainerGallery = !!number.match(/^[tg]g/i) || !!( id === "swshp-SWSH076" || id === "swshp-SWSH077" );
    if (Array.isArray(types)) {
      types = types.join(" ").toLowerCase();
    }
    if (Array.isArray(subtypes)) {
      subtypes = subtypes.join(" ").toLowerCase();
    }
  }

</script>

<div
  class="card {types} / interactive / "
  class:active
  class:interacting
  class:loading
  class:masked={!!mask}
  data-number={number}
  data-set={set}
  data-subtypes={subtypes}
  data-supertype={supertype}
  data-rarity={rarity}
  data-trainer-gallery={isTrainerGallery}
  style={dynamicStyles}
  bind:this={thisCard}
>
  <div 
    class="card__translater">
    <button
      class="card__rotator"
      on:pointermove={interact}
      on:mouseout={interactEnd}
      on:blur={interactEnd}
      aria-label="Expand the Pokemon Card; {name}."
      tabindex="0"
      >
      <img
        class="card__back"
        src={back_img}
        alt="The back of a Pokemon Card, a Pokeball in the center with Pokemon logo above and below"
        loading="lazy"
        width="660"
        height="921"
      />
      <div class="card__front" 
        style={ staticStyles + foilStyles }>
        <img
          src={img_base + img}
          alt="Front design of the {name} Pokemon Card, with the stats and info around the edge"
          loading="lazy"
          width="660"
          height="921"
        />
        <div class="card__shine"></div>
        <div class="card__glare"></div>
      </div>
    </button>
  </div>
</div>

<style>

  :root {
    --pointer-x: 50%;
    --pointer-y: 50%;
    --card-scale: 1;
    --card-opacity: 0;
    --translate-x: 0px;
    --translate-y: 0px;
    --rotate-x: 0deg;
    --rotate-y: 0deg;
    --background-x: var(--pointer-x);
    --background-y: var(--pointer-y);
    --pointer-from-center: 0;    
    --pointer-from-top: var(--pointer-from-center);
    --pointer-from-left: var(--pointer-from-center);
  }

</style>
