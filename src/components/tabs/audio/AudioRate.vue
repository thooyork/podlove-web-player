<template>
  <div class="input-element">
    <h4 class="title">
      <span class="label">{{ $t('AUDIO.SPEED') }}</span>
      <span class="rate">{{ toPercent(rate) }}%</span>
    </h4>
    <div class="input-slider">
      <ButtonComponent class="slider-button" :click="changeRate(-5, rate)" :style="buttonStyle">
        <MinusIcon :color="theme.tabs.button.text"></MinusIcon>
      </ButtonComponent>
      <ButtonComponent class="slider-button" :click="changeRate(5, rate)" :style="buttonStyle">
        <PlusIcon :color="theme.tabs.button.text"></PlusIcon>
      </ButtonComponent>
      <SliderComponent class="input-slider"
        min="0" max="1" step="0.001"
        :value="sliderRate" :onInput="toStateRate" :thumbBorder="theme.tabs.input.border" :thumbColor="theme.tabs.slider.thumb"></SliderComponent>
    </div>
  </div>
</template>

<script>
  import store from 'store'

  import { compose } from 'lodash/fp'
  import { toPercent, roundUp, round } from 'utils/math'

  import SliderComponent from 'shared/Slider.vue'
  import ButtonComponent from 'shared/Button.vue'

  import PlusIcon from 'icons/PlusIcon.vue'
  import MinusIcon from 'icons/MinusIcon.vue'

  // Speed Modifiers
  const normalizeSliderValue = (value = 0) => {
    if (value < 0) {
      value = 0
    }

    if (value > 1) {
      value = 1
    }

    return value
  }

  const normalizeRateValue = (value = 0) => {
    if (value < 0.5) {
      value = 0.5
    }

    if (value > 4) {
      value = 4
    }

    return value
  }

  const speedSliderToState = (value = 0) => {
    value = parseFloat(value)

    if (value <= 0.5) {
      value = 0.5 + value
    } else {
      value = 2 * value + (value - 0.5) * 4
    }

    return value
  }

  const stateToSpeedSlider = (value = 0) => {
    value = parseFloat(value)

    if (value <= 1) {
      value = value - 0.5
    } else {
      value = (value + 2) / 6
    }

    return value
  }

  // State Changers
  const setRate = compose(store.dispatch.bind(store), store.actions.setRate)
  const toStateRate = compose(setRate, round, speedSliderToState, normalizeSliderValue)
  const toSliderRate = compose(round, stateToSpeedSlider, normalizeRateValue)

  const changeRate = (offset, rate) => () => compose(setRate, roundUp(offset))(rate)

  export default {
    data () {
      return {
        rate: this.$select('rate'),
        theme: this.$select('theme')
      }
    },
    computed: {
      sliderRate: function () {
        return toSliderRate(this.rate)
      },
      buttonStyle () {
        return {
          color: this.theme.tabs.button.text,
          background: this.theme.tabs.button.background,
          'border-color': this.theme.tabs.input.border
        }
      }
    },
    methods: {
      setRate,
      toStateRate,
      toSliderRate,
      changeRate,
      toPercent
    },
    components: {
      SliderComponent,
      ButtonComponent,
      PlusIcon,
      MinusIcon
    }
  }
</script>

<style lang="scss">
  @import 'variables';
  @import 'inputs';

  .audio {
    .title {
      display: flex;
      justify-content: space-between;
    }
  }
</style>