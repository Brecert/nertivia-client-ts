<template>
  <div class="message-embed">
    <ImageEmbed v-if="imageEmbed" :image="imageEmbed" />
    <YoutubeEmbed v-else-if="youtubeEmbed" :embed="embed" />
    <GenericEmbed v-else :embed="embed" />
  </div>
</template>

<script lang="ts">
import { Embed } from "@/interfaces/Message";
import { Component, Prop, Vue } from "vue-property-decorator";
import GenericEmbed from "./GenericEmbed.vue";
import ImageEmbed from "./ImageEmbed.vue";
import YoutubeEmbed from "./YoutubeEmbed.vue";

@Component({ components: { GenericEmbed, ImageEmbed, YoutubeEmbed } })
export default class EmbedMessage extends Vue {
  @Prop() private embed!: Embed;

  get youtubeEmbed() {
    if (this.embed.site_name !== "YouTube") return false;
    if (this.embed.type !== "video.other") return false;
    const regex = /((http(s)?:\/\/)?)(www\.)?((youtube\.com\/)|(youtu.be\/))[\S]+/gm;
    if (!this.embed.url.match(regex)) return false;
    return true;
  }

  get imageEmbed() {
    if (Object.keys(this.embed).length !== 1) return undefined;
    if (!this.embed.image) return undefined;
    return this.embed.image;
  }
}
</script>
<style lang="scss" scoped>
.message-embed {
  flex-direction: column;
  flex-shrink: 0;
  margin-left: 56px;
  margin-top: 5px;
  border-radius: 4px;
  background: rgba(0, 0, 0, 0.4);
  align-self: flex-start;
  overflow: hidden;
}
</style>
