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
      this.$nextTick(() => {
        const webview = this.$refs[`webview-${extension.extensionId}`][0];
        webview.addEventListener('context-menu', (event) => {
          const { Menu, MenuItem } = (this as any).$electron.remote;
          const menu = new Menu();

          menu.append(new MenuItem({
            label: 'Inspect Element',
            click: () => {
              webview.inspectElement(event.params.x, event.params.y);
            },
          }));

          menu.popup((this as any).$electron.remote.getCurrentWindow(), { async: true });
        });
        webview.addEventListener('ipc-message', (event: Electron.IpcMessageEvent) => {
          if (event.channel === 'resize') {
            const size = event.args[0];
            webview.style.height = `${size.height}px`;
            webview.style.width = `${size.width}px`;
            webview.style.overflow = 'hidden';
          }
        });
        webview.addEventListener('dom-ready', () => {
          webview.executeJavaScript(`
            function triggerResize() {
              const height = document.body.clientHeight;
              const width = document.body.clientWidth;
              ipcRenderer.sendToHost('resize', {
                height,
                width,
              });
            }
            triggerResize();
            new ResizeSensor(document.body, triggerResize);
          `);
        });
      });
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
