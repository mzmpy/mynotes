<template>
  <a-layout :style="{ height: '100vh', padding: '10px 36px 0' }">
    <a-layout-header class="header">
      <div class="logo" />
      <a-menu theme="dark" mode="horizontal" v-model:selectedKeys="selectedKeys" :style="{ lineHeight: '64px' }">
        <a-menu-item key="1">
          <span class="nav-text">nav 1</span>
        </a-menu-item>
        <a-menu-item key="2">
          <span class="nav-text">nav 2</span>
        </a-menu-item>
        <a-menu-item key="3">
          <span class="nav-text">nav 3</span>
        </a-menu-item>
      </a-menu>
    </a-layout-header>
    <a-layout-content :style="{ overflow: 'auto' }">
      <!-- Note: 这里的paddingTop不能替换为marginTop，否则就会出现scrollBar，这与盒子的高度继承有关 -->
      <a-layout :style="{ height: '100%', paddingTop: '13px' }">
        <a-layout-sider
         :style="sider"
         :width="siderWidth"
         >
          <div class="after-sider" @mousedown.prevent="changeSiderWidth"></div>
         </a-layout-sider>
        <a-layout-content :style="{ height: '100%', overflow: 'auto', marginLeft: '10px' }">
          <div :style="{ overflow: 'auto', height: '100%', background: '#fff' }">
            <div
             :style="{ width: '100%', height: '50%', overflow: 'hidden', padding: '15px 15px 0 15px' }">
              <div
              class="markdown-body"
               ref="markdown"
               :style="{ width: '100%', height: '100%', border: '1px solid black', overflow: 'auto', background: '#fff', textAlign: 'left', padding: '15px' }">
              </div>
            </div>
            <monaco-editor :style="{ height: '50%', padding: '15px' }" @codeChanged="onCodeChanged"></monaco-editor>
          </div>
        </a-layout-content>
      </a-layout>
    </a-layout-content>
    <div class="footer-btn" :style="divDisplay" @click="footerBtnHlr"></div>
    <a-layout-footer :style="footer" @click="footerHlr">
      <hr>
      MyNotes ©2021 Created by Ma Zhiming
    </a-layout-footer>
  </a-layout>
</template>

<script>
  import { defineComponent, ref } from 'vue';
  import MonacoEditor from '../editor/editor';
  import hljs from 'highlight.js';
  import 'highlight.js/styles/github.css';
  import marked from 'marked';
  import 'github-markdown-css/github-markdown.css';

  export default defineComponent({
    data() {
      return {
        footer: {
          display: 'none',
          margin: '10px 0',
          padding: 0,
          textAlign: 'center',
        },
        divDisplay: {
          display: 'block',
        },
        sider: {
          height: '100%',
          overflow: 'auto',
          backgroundColor: '#ffffff',
          position: 'relative',
        },
        siderWidth: '300px',
        dragStartX: null,
      }
    },

    setup() {
      return {
        selectedKeys: ref(['4']),
        menuCount: 7,
      };
    },

    components: {
      MonacoEditor,
    },

    methods: {
      footerBtnHlr() {
        this.footer.display = 'block';
        this.divDisplay.display = 'none';
      },

      footerHlr() {
        this.footer.display = 'none';
        this.divDisplay.display = 'block';
      },

      onMouseMove(moveEvent) {
        new Promise((resolve) => {
          resolve(moveEvent.clientX);
        }).then((dragCurrentX) => {
          this.siderWidth = (parseFloat(this.siderWidth) + (dragCurrentX - this.dragStartX)) + 'px';
          this.dragStartX = dragCurrentX;
        }).catch((err) => {
          console.log(err);
        });
      },

      onMouseUp() {
        document.removeEventListener('mousemove', this.onMouseMove);
        document.removeEventListener('mouseup', this.onMouseUp);
      },

      changeSiderWidth(downEvent) {
        this.dragStartX = downEvent.clientX;

        document.addEventListener('mousemove', this.onMouseMove);
        document.addEventListener('mouseup', this.onMouseUp);
      },

      markIt(val) {
        marked.setOptions({
          renderer: new marked.Renderer(),
          highlight: function(code, lang) {
            const language = hljs.getLanguage(lang) ? lang : 'python';
            return hljs.highlight(code, { language }).value;
          },
          langPrefix: 'hljs language-',
          pedantic: false,
          gfm: true,
          breaks: false,
          sanitize: false,
          smartLists: true,
          smartypants: false,
          xhtml: false
        });

        this.$refs.markdown.innerHTML = marked(val);
        /* document.querySelectorAll('pre code').forEach((el) => {
          hljs.highlightElement(el);
        }); */
      },

      onCodeChanged(val) {
        this.markIt(val);
      },
    },
  });
</script>

<style>
  .logo {
    float: left;
    width: 25%;
    height: 32px;
    background: rgba(212, 212, 212, 0.2);
    margin: 16px 24px 16px 0;
  }

  .footer-btn {
    height: 24px;
  }

  .after-sider {
    display: block;
    position: absolute;
    right: 0;
    top: 0;
    height: 100%;
    width: 3px;
    background-color: #ffffff;
  }

  .after-sider:hover {
    background: linear-gradient(to right, rgb(211, 211, 211), #ffffff);
    cursor: e-resize;
  }

</style>