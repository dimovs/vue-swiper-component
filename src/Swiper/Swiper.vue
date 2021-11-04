<template>
  <div />
</template>

<script>
import { h, ref, computed } from 'vue'
import SwiperCore, { Autoplay, Pagination } from 'swiper/core'
SwiperCore.use([Autoplay, Pagination])
const { Swiper, SwiperSlide } = require('swiper/vue')

export default {
  name: 'Swiper',
  props: {
    options: {
      type: Object,
      default: () => ({}),
    },
    maxTotalItems: {
      type: Object,
      default: () => ({
        mobile: 4,
        tablet: 8,
      }),
    },
  },
  setup (props, { slots }) {
    const BREAKPOINTS = {
      MOBILE: '320',
      TABLET: '768',
      DESKTOP: '1112',
    }

    const defaultOptions = {
      direction: 'horizontal',
      breakpoints: {
        [BREAKPOINTS.MOBILE]: {
          slidesPerView: 1,
          slidesPerGroup: 1,
        },
        [BREAKPOINTS.TABLET]: {
          slidesPerView: 2,
          slidesPerGroup: 2,
        },
      },
      slidesPerView: 1,
      slidesPerGroup: 1,
      slidesSpacing: 0,
      loop: false,
      speed: 3000,
      autoPlay: {
        delay: 3000,
        disableOnInteraction: false,
        pauseOnMouseEnter: true,
      },
      grabCursor: true,
    }

    const mergedOptions = Object.assign(defaultOptions, props.options)
    const currentMaxTotalItems = ref(props.maxTotalItems.tablet)
    const listItems = computed(() => slots.default()?.find(slot => typeof slot.type === 'symbol')?.children || slots.default())
    const totalItems = computed(() => listItems.value
        .filter((item, index) => index < currentMaxTotalItems.value)
        .map(item => h(SwiperSlide, null, { default: () => item })),
    )

    const updateSwiperItemsLength = (itemsLength, callBack) => {
      if (itemsLength) {
        callBack()
        currentMaxTotalItems.value = itemsLength
      }
    }

    const classes = [
      mergedOptions.fullHeight ? 'full-height' : '',
    ]

    return () => {
      return h('div', {}, [
        h(Swiper,
            {
              ...mergedOptions,
              class: classes,
              pagination: {
                el: '.swiper-pagination-custom',
                clickable: true,
                renderBullet (index, className) {
                  return `<div class="${className} swiper-pagination-bullet-custom"></div>`
                },
              },
              onInit: swiper => {
                if (swiper.currentBreakpoint === BREAKPOINTS.MOBILE) {
                  updateSwiperItemsLength(props.maxTotalItems?.mobile, swiper.update)
                }
              },
              onBeforeResize: swiper => {
                const itemsLength = swiper.currentBreakpoint === BREAKPOINTS.MOBILE
                    ? props.maxTotalItems?.mobile
                    : props.maxTotalItems?.tablet

                updateSwiperItemsLength(itemsLength, swiper.update)
              },
            },
            {
              default: () => [
                totalItems.value,
                h('div', {
                  class: ['swiper-pagination-custom'],
                }),
              ],
            },
        ),
      ])
    }
  },
}
</script>
<style lang="scss">
@import '~swiper/swiper-bundle.css';

.swiper-container {
  &.full-height {
    .swiper-wrapper {
      display: flex;
      align-items: stretch;

      .swiper-slide {
        height: auto;
      }
    }
  }
}

.swiper-container-horizontal {
  .swiper-pagination-custom {
    display: flex;
    align-items: center;
    justify-content: center;
    caret-color: transparent;

    .swiper-pagination-bullet-custom {
      background: none;
      position: relative;
      opacity: 0.5;
      width: 24px;
      height: 8px;
      cursor: pointer;
      display: flex;
      justify-content: center;
      align-items: center;
      margin: 0;

      &::before {
        position: absolute;
        display: block;
        content: '';
        width: 16px;
        height: 4px;
        border-radius: 2px;
      }
    }

    .swiper-pagination-bullet-active {
      opacity: 1;
    }
  }
}
</style>
