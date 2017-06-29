<template>
  <transition appear name="v-img-fade">
    <v-touch @pan="pan" @panend="panEnd" @pinch="pinch" @pinchend="pinchEnd">
      <div
        v-if="!closed"
        class='fullscreen-v-img'
        @click.self="close()" ref="screen">
        <!--
          Count of total images in array, current position.
          Hidden if only one image
        -->
        <div class="header-v-img">
          <span
            v-if="images.length > 1"
            class="count-v-img">
            {{ currentImageIndex + 1 }}/{{ images.length }}
          </span>
          <span
            class="close-v-img"
            @click="close">
            &times;
          </span>
        </div>
        <!-- Controls start -->
        <transition appear name="v-img-fade">
          <span
            v-if="visibleUI && images.length !== 1"
            class="prev-v-img"
            @click="prev">
            <svg width="25" height="25" viewBox="0 0 1792 1915" xmlns="http://www.w3.org/2000/svg"><path d="M1664 896v128q0 53-32.5 90.5t-84.5 37.5h-704l293 294q38 36 38 90t-38 90l-75 76q-37 37-90 37-52 0-91-37l-651-652q-37-37-37-90 0-52 37-91l651-650q38-38 91-38 52 0 90 38l75 74q38 38 38 91t-38 91l-293 293h704q52 0 84.5 37.5t32.5 90.5z" fill="#fff"/></svg>
          </span>
        </transition>
        <transition appear name="v-img-fade">
          <span
            v-if="visibleUI && images.length !== 1"
            class="next-v-img"
            @click="next">
            <svg width="25" height="25" viewBox="0 0 1792 1915" xmlns="http://www.w3.org/2000/svg"><path d="M1600 960q0 54-37 91l-651 651q-39 37-91 37-51 0-90-37l-75-75q-38-38-38-91t38-91l293-293h-704q-52 0-84.5-37.5t-32.5-90.5v-128q0-53 32.5-90.5t84.5-37.5h704l-293-294q-38-36-38-90t38-90l75-75q38-38 90-38 53 0 91 38l651 651q37 35 37 90z" fill="#fff"/></svg>
          </span>
        </transition>
        <!-- Constols end -->

        <div class="content-v-img">
          <transition appear name="v-img-fade">
            <img :src="images[currentImageIndex]" @click="next" ref="image">
          </transition>
        </div>
      </div>
    </v-touch>
  </transition>
</template>

<script>
  export default {
    data() {
      return {
        images: [],
        visibleUI: true,
        currentImageIndex: 0,
        closed: true,
        uiTimeout: null,
        state: {
          scale: 1,
          translate: [0, 0]
        },
      }
    },
    methods: {
      pinch (ev) {
        const scale = this.calScale(ev.scale)
        this.transform({ translate: this.state.translate, scale: scale })
      },
      pinchEnd (ev) {
        this.state.scale = this.calScale(ev.scale)
      },
      pan (ev) {
        const ts = this.calTranslate([ev.deltaX, ev.deltaY])
        this.transform({ translate: ts, scale: this.state.scale })
      },
      panEnd (ev) {
        this.state.translate = this.calTranslate([ev.deltaX, ev.deltaY])
      },
      calScale (s) {
        let scale = this.state.scale * s
        scale = scale < 5 ? scale : 5
        scale = scale > 1 ? scale : 1
        return scale
      },
      calTranslate (translate) {
        const viewport = {
          w: this.$refs.screen.offsetWidth,
          h: this.$refs.screen.offsetHeight
        }
        let ts = [
          this.state.translate[0] + translate[0],
          this.state.translate[1] + translate[1]
        ]

        const sw = this.$refs.image.width * this.state.scale
        const sh = this.$refs.image.height * this.state.scale
        const maxWidthRange = (sw - viewport.w) / 2
        const maxHeightRange = (sh - viewport.h) / 2

        if (sw < viewport.w) ts[0] = 0
        else {
          if (maxWidthRange > 0) {
            const x = ts[0]

            if (x > 0) {
              ts[0] = Math.abs(x) < maxWidthRange ? x : maxWidthRange
            } else {
              ts[0] = Math.abs(x) < maxWidthRange ? x : -maxWidthRange
            }
          } else {
            ts[0] = 0
          }
        }

        if (sh < viewport.h) ts[1] = 0
        else {
          if (maxHeightRange > 0) {
            const y = ts[1]

            if (y > 0) {
              ts[1] = Math.abs(y) < maxHeightRange ? y : maxHeightRange
            } else {
              ts[1] = Math.abs(y) < maxHeightRange ? y : -maxHeightRange
            }
          } else {
            ts[1] = 0
          }
        }

        return ts
      },
      transform (state) {
        const { translate: ts, scale } = state
        this.$refs.image.style.transform = `translate(-50%, -50%) translate(${ts[0]}px, ${ts[1]}px) scale(${scale})`
      },
      close() {
        document.querySelector('body').classList.remove('body-fs-v-img');
        this.images = [];
        this.currentImageIndex = 0;
        this.closed = true;
        this.initState()
      },
      next() {
        this.initState()
        // if next index not exists in array of images, set index to first element
        if (this.currentImageIndex + 1 < this.images.length) {
          this.currentImageIndex++;
        } else {
          this.currentImageIndex = 0;
        };
      },
      prev() {
        this.initState()
        // if prev index not exists in array of images, set index to last element
        if (this.currentImageIndex > 0) {
          this.currentImageIndex--;
        } else {
          this.currentImageIndex = this.images.length - 1;
        };
      },
      showUI(){
        // UI's hidden, we reveal it for some time only on mouse move and
        // ImageScreen appear
        clearTimeout(this.uiTimeout);
        this.visibleUI = true;
        this.uiTimeout = setTimeout(() => {
          this.visibleUI = false
        }, 3500)
      },
      initState () {
        this.state = { scale: 1, translate: [0, 0] }
        this.transform(this.state)
      }
    },
    created() {
      window.addEventListener('keyup', (e) => {
        // esc key and 'q' for quit
        if (e.keyCode === 27 || e.keyCode === 81) this.close();
        // arrow right and 'l' key (vim-like binding)
        if (e.keyCode === 39 || e.keyCode === 76) this.next();
        // arrow left and 'h' key (vim-like binding)
        if (e.keyCode === 37 || e.keyCode === 72) this.prev();
      });
      window.addEventListener('scroll', () => {
        this.close();
      });
      window.addEventListener('mousemove', () => {
        this.showUI();
      });
    },
  };
</script>

<style>
  .body-fs-v-img {
  }
</style>

<style scoped>

  * {
    -webkit-box-sizing: border-box;
            box-sizing: border-box;
  }

  .fullscreen-v-img {
    height: 100%;
    width: 100%;
    position: fixed;
    top: 0;
    left: 0;
    overflow: hidden;
    background-color: rgba(0, 0, 0, .7);
    -ms-touch-action: none;
        touch-action: none;
  }

  .content-v-img img {
    position: absolute;
    max-width: 100%;
    max-height: 100%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    -webkit-user-select: none;
       -moz-user-select: none;
        -ms-user-select: none;
            user-select: none;
  }

  .header-v-img {
    position: absolute;
    width: 100%;
    background-color: rgba(0, 0, 0, .3);
    height: 50px;
    z-index: 9999;
  }

  .count-v-img {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 15px;
    color: white;
    margin-left: 10px;
    line-height: 50px;
  }

  .close-v-img {
    float: right;
    margin-right: 10px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    color: #E5E6EB;
    font-size: 30px;
    line-height: 50px;
    cursor: pointer;
    -webkit-transition: color .4s ease-in-out;
    transition: color .4s ease-in-out;
  }

  .close-v-img:hover {
    color: white;
  }

  .prev-v-img svg,
  .next-v-img svg {
    margin: 5px auto;
  }

  .prev-v-img,
  .next-v-img {
    color: white;
    width: 35px;
    height: 35px;
    position: absolute;
    top: 50%;
    margin-top: -12.5px;
    font-size: 15px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    text-align: center;
    background-color: rgba(0, 0, 0, .3);
    z-index: 1000;
    opacity: .3;
    -webkit-transition: opacity .3s ease-in-out;
    transition: opacity .3s ease-in-out;
    cursor: pointer;
  }

  .prev-v-img:hover,
  .next-v-img:hover {
    opacity: 1;
  }

  .prev-v-img {
    left: 10px;
  }

  .next-v-img {
    right: 10px;
  }

  .v-img-fade-enter,
  .v-img-fade-leave-to {
    opacity: 0;
  }

  .v-img-fade-enter-active,
  .v-img-fade-leave-active {
    -webkit-transition: opacity .3s ease-in-out;
    transition: opacity .3s ease-in-out;
  }
</style>
