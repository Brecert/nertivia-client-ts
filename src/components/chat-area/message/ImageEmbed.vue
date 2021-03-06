<template>
  <div
    class="image-embed"
    :class="{ animate: isWindowFocused && !loadImage }"
    @click="onClick"
  >
    <div class="outer-content">
      <div class="inner-content" ref="content">
        <div class="gif" v-if="isGif">GIF</div>
        <img v-if="loadImage" :src="pauseGifURL" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";
import windowProperties from "@/utils/windowProperties";
import { PopoutsModule } from "@/store/modules/popouts";
@Component
export default class ImageMessageEmbed extends Vue {
  @Prop() private image!: any;
  loadImage = false;
  intersectObserver: IntersectionObserver | null = null;

  onClick() {
    PopoutsModule.ShowPopout({
      id: "image-preview-popout",
      component: "image-preview-popout",
      data: {
        url: this.imageURL
      }
    });
  }

  mounted() {
    const contentEl = this.$refs["content"] as any;
    this.intersectObserver = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.fetchImage();
          this.intersectObserver?.unobserve(contentEl);
          this.intersectObserver?.disconnect();
        }
      });
    });
    this.intersectObserver.observe(contentEl);
  }
  fetchImage() {
    const image = new Image();
    image.onload = () => {
      image.onload = null;
      this.loadImage = true;
    };
    image.src = this.pauseGifURL || "";
  }

  get isWindowFocused() {
    return windowProperties.isFocused;
  }
  // pause gif when window is not focused
  get pauseGifURL() {
    let url = this.imageURL;
    if (!this.isWindowFocused && this.isGif) {
      url += `?type=webp`;
    }
    return url;
  }
  get isGif() {
    return this.imageURL?.endsWith(".gif");
  }
  get imageURL() {
    return (
      process.env.VUE_APP_IMAGE_PROXY_URL + encodeURIComponent(this.image.url)
    );
  }
}
</script>

<style scoped lang="scss">
.image-embed {
  position: relative;
  border-radius: 4px;
  background: rgba(0, 0, 0, 0.4);
  overflow: hidden;

  cursor: pointer;
  &:hover {
    &::after {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      pointer-events: none;
      border-radius: 4px;
      overflow: hidden;
      background: rgba(0, 0, 0, 0.3);
    }
  }
}
.inner-content {
  display: flex;
}
.outer-content {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 200px;
}

img {
  display: grid;

  height: auto;

  min-width: 200px;
  min-height: 200px;

  max-width: min(500px, 100%);
  max-height: 500px;

  object-fit: contain;
}

.image-embed:hover {
  .gif {
    opacity: 0;
  }
}
.gif {
  position: absolute;
  top: 5px;
  left: 5px;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(5px);
  padding: 3px;
  border-radius: 4px;
  transition: 0.2s;
}
.animate {
  animation: animate 1s;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}
@keyframes animate {
  from {
    background: rgba(0, 0, 0, 0.4);
  }
  to {
    background: rgba(0, 0, 0, 0.5);
  }
}
</style>
