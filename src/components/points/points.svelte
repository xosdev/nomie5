<script>
  export let points = 0;
  export let className = undefined;
  $: dir = points > 0 ? "+" : points < 0 ? "" : "";
  $: positiveClass = points > 0 ? "positive" : points < 0 ? "negative" : "neutral";
</script>

<style lang="postcss">
  .n-points {
	 position: relative;
	 border-radius: 13px;
	 display: flex;
	 align-items: center;
	 justify-content: center;
	 font-size: 1em;
	 font-weight: bolder;
	 overflow: visible;
	 max-height: 22px;
}
 .n-points.positive {
	 color: var(--color-green);
}
 .n-points.positive .dir {
	 color: var(--color-green);
}
 .n-points svg {
	 margin-left: 2px;
	 position: absolute;
	 display: block;
	 width: 12px;
	 height: 12px;
	 right: -7px;
	 top: -4px;
}
 .n-points.negative {
	 color: var(--color-red);
}
 .n-points.negative .dir {
	 color: var(--color-red);
}
 .n-points.neutral {
	 color: var(--color-primary-bright);
}
 .n-points.neutral .dir {
	 color: var(--color-primary-bright);
}
 
</style>

{#if points !== 0}
  <div class="n-points {className} {positiveClass}">
    <span class="dir">{dir}</span>
    <span class="number">{points}</span>
    {#if points > 0}
      <svg width="19" height="19" viewBox="0 0 19 19" fill="none" xmlns="http://www.w3.org/2000/svg">
        <g clip-path="url(#clip0)">
          <path
            d="M-4.15258e-07 9.5C-6.44599e-07 14.7467 4.25329 19 9.5 19C14.7467 19 19 14.7467 19 9.5C19 4.25329 14.7467 -1.85917e-07 9.5
            -4.15258e-07C4.25329 -6.44599e-07 -1.85917e-07 4.25329 -4.15258e-07 9.5Z"
            fill="#FFD93B" />
          <path
            d="M9.4999 15.606C6.72175 15.606 4.30103 13.7244 3.61347 11.0312C3.55943 10.8192 3.68709 10.6037 3.89906 10.5497C4.10984 10.4944
            4.32597 10.6233 4.38 10.8352C4.9785 13.1782 7.08334 14.8146 9.49931 14.8146C11.9153 14.8146 14.0207 13.1782 14.6186
            10.8352C14.6721 10.6233 14.8876 10.4944 15.1002 10.5497C15.3121 10.6037 15.4398 10.8192 15.3857 11.0312C14.6988 13.7244 12.2781
            15.606 9.4999 15.606Z"
            fill="#3E4347" />
          <path
            d="M6.43195 9.90569C7.23601 9.90569 7.88782 9.13106 7.88782 8.1755C7.88782 7.21994 7.23601 6.44531 6.43195 6.44531C5.62789
            6.44531 4.97607 7.21994 4.97607 8.1755C4.97607 9.13106 5.62789 9.90569 6.43195 9.90569Z"
            fill="#3E4347" />
          <path
            d="M7.08871 7.8032C6.9753 7.89227 6.78886 7.84833 6.66952 7.70345C6.55017 7.55858 6.54424 7.36442 6.65764 7.27536C6.77046
            7.18214 6.95749 7.22608 7.07624 7.37096C7.19618 7.51999 7.19736 7.71058 7.08871 7.8032Z"
            fill="#5A5F63" />
          <path
            d="M12.5696 9.90569C13.3737 9.90569 14.0255 9.13106 14.0255 8.1755C14.0255 7.21994 13.3737 6.44531 12.5696 6.44531C11.7656
            6.44531 11.1138 7.21994 11.1138 8.1755C11.1138 9.13106 11.7656 9.90569 12.5696 9.90569Z"
            fill="#3E4347" />
          <path
            d="M13.2249 7.8032C13.1115 7.89227 12.9251 7.84833 12.8057 7.70345C12.6864 7.55858 12.6805 7.36442 12.7939 7.27536C12.9067
            7.18214 13.0931 7.22608 13.2125 7.37096C13.3324 7.51999 13.3336 7.71058 13.2249 7.8032Z"
            fill="#5A5F63" />
        </g>
        <defs>
          <clipPath id="clip0">
            <rect width="19" height="19" fill="white" />
          </clipPath>
        </defs>
      </svg>
    {:else}
      <svg width="19" height="19" viewBox="0 0 19 19" fill="none" xmlns="http://www.w3.org/2000/svg">
        <g clip-path="url(#clip0)">
          <path
            d="M-4.15258e-07 9.5C-6.44599e-07 14.7467 4.25329 19 9.5 19C14.7467 19 19 14.7467 19 9.5C19 4.25329 14.7467 -1.85917e-07 9.5
            -4.15258e-07C4.25329 -6.44599e-07 -1.85917e-07 4.25329 -4.15258e-07 9.5Z"
            fill="#FFB13B" />
          <path
            d="M13.1586 15.1102C12.686 15.3952 11.4142 14.127 9.66145 14.1115C7.93423 14.0961 6.63689 15.3958 6.16427 15.1102C5.59842
            14.8799 6.56623 12.1189 9.66145 12.1023C12.7567 12.1189 13.7245 14.8805 13.1586 15.1102Z"
            fill="#1C1C1C" />
          <path
            d="M14.485 8.22031C14.485 9.0569 13.8005 9.74149 12.9603 9.74149C12.1195 9.74149 11.4355 9.0569 11.4355 8.22031C11.4355 7.37599
            12.1201 6.69556 12.9603 6.69556C13.801 6.69556 14.485 7.37599 14.485 8.22031Z"
            fill="#19191A" />
          <path
            d="M13.6584 7.36028C13.471 7.19754 13.235 7.16236 13.1314 7.28171C13.0278 7.40106 13.0957 7.62974 13.2832 7.79249C13.4706
            7.95523 13.7066 7.99041 13.8102 7.87106C13.9138 7.75171 13.8459 7.52303 13.6584 7.36028Z"
            fill="#5A5F63" />
          <path
            d="M7.56415 8.22031C7.56415 9.0569 6.87955 9.74149 6.0394 9.74149C5.19924 9.74149 4.51465 9.0569 4.51465 8.22031C4.51465 7.37599
            5.19924 6.69556 6.0394 6.69556C6.87955 6.69556 7.56415 7.37599 7.56415 8.22031Z"
            fill="#19191A" />
          <path
            d="M6.73703 7.36077C6.54958 7.19803 6.31363 7.16285 6.21001 7.2822C6.10638 7.40155 6.17433 7.63023 6.36178 7.79297C6.54922
            7.95572 6.78517 7.99089 6.8888 7.87155C6.99242 7.7522 6.92447 7.52352 6.73703 7.36077Z"
            fill="#5A5F63" />
        </g>
        <defs>
          <clipPath id="clip0">
            <rect width="19" height="19" fill="white" />
          </clipPath>
        </defs>
      </svg>
    {/if}

  </div>
{/if}
