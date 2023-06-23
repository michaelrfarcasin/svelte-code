<script lang="ts">
  import { onMount } from 'svelte';
  
  export let commaSeparatedValues: string[] = [];
  let maxWidth: number = 0;
  let numberToShow: number = 1;
  let valuesWidth: number = 0;
  let remainder: number = 0;
  let formattedValues: string = '';

  const remainderPadding: number = 5;

  const initializeDisplay = (): void => {
    numberToShow = Math.max(1, getNumberOfValuesThatFit());
    valuesWidth = maxWidth - (2 * remainderPadding) - calculateTextWidth('+' + numberToShow);
    remainder = commaSeparatedValues.length - numberToShow;
    formattedValues = formatValues(numberToShow);
  }

  const getNumberOfValuesThatFit = (): number => {
    let numberToShow: number = 0;
    commaSeparatedValues.some((element, index) => {
      let text = formatValues(index + 1);
      let textWidth = 0;
      if (index < commaSeparatedValues.length) {
        text += '+' + (commaSeparatedValues.length - index);
        textWidth = remainderPadding * 2;
      }
      textWidth += calculateTextWidth(text);
      if (textWidth > maxWidth) {
        return true;
      } else {
        numberToShow++;
      }
    });
    
    return numberToShow;
  }

  const formatValues = (numberOfValues: number): string => {
    let text = joinValues(numberOfValues);
    if (numberOfValues < commaSeparatedValues.length) {
      if (commaSeparatedValues.length > 1) {
          text += ', ';
      }
      text += '...';
    }

    return text;
  }

  const joinValues = (numberOfValues: number): string => {
    return commaSeparatedValues.slice(0, numberOfValues).join(', ');
  }
  
  const calculateTextWidth = (text: string): number => {
    var tag = document.createElement('div');
    tag.style.position = 'absolute';
    tag.style.whiteSpace = 'nowrap';
    tag.innerHTML = text;

    document.body.appendChild(tag);
    var result = tag.clientWidth;
    document.body.removeChild(tag);

    return result;
  }

  const debounce = (func: Function, timeout: number) => {
    let timeoutID: number;
    
    return (...args: any[]): void => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => { func.apply(this, args); }, timeout);
    };
  }

  onMount(initializeDisplay);

  const initializeDisplayAfterDelay = debounce(initializeDisplay, 10);
</script>

<style>
  .values {
    display: block;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: var(--values-width);
  }

  .remainder {
    position: absolute;
    right: 0;
    top: 50%;
    transform: translateY(-50%);
    font-size: 16px;
    color: #f0f0f0;
    background-color: #666666;
    border-radius: 3px;
    padding: 2px var(--remainder-padding);
  }
</style>

<svelte:window on:resize={initializeDisplayAfterDelay} />

<div bind:clientWidth={maxWidth}>
  <span class='values' style="--values-width: {valuesWidth}px">
    {formattedValues}
  </span>
  {#if remainder > 0}
    <span class='remainder' style="--remainder-padding: {remainderPadding}px">+{remainder}</span>
  {/if}
</div>
