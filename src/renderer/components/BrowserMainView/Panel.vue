<template lang="pug">
  #browser-panel
    div(v-for="extension in extensions",
        :key="extension.extensionId")
      webview.extension(:src="buildPanelPath(extension)",
                        :ref="`webview-${extension.extensionId}`")
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator';

  import url from 'url';

  @Component({
    props: [
      'windowId',
    ],
  })
  export default class Panel extends Vue {
    extensions: any[] = [];

    windowId: number;

    buildPanelPath(extension) {
      return url.format({
        protocol: 'lulumi-extension',
        slashes: true,
        hostname: extension.extensionId,
        pathname: extension.panel,
      });
    }

    mounted() {
      // tslint:disable-next-line
      console.log(this.extensions);
    }
  };
</script>

<style lang="less" scoped>
  #browser-panel {
    display: flex;
    flex-direction: column;
    height: auto;
    padding: 0 5px;
    border-bottom: 1px solid #888;
  }
</style>
