<template>
  <nav :class="{ 'search-box-open': isSearchBoxOpen }">
    <div class="win32-titlebar">
      <div class="title">YesPlayMusic</div>
      <div class="controls">
        <div
          class="button minimize codicon codicon-chrome-minimize"
          @click="windowMinimize"
        ></div>
        <div
          class="button max-restore codicon"
          @click="windowMaxRestore"
          :class="{
            'codicon-chrome-restore': windowIsMaximized,
            'codicon-chrome-maximize': !windowIsMaximized,
          }"
        ></div>
        <div
          class="button close codicon codicon-chrome-close"
          @click="windowClose"
        ></div>
      </div>
    </div>
    <div class="navigation-buttons">
      <button-icon @click.native="go('back')"
        ><svg-icon icon-class="arrow-left"
      /></button-icon>
      <button-icon @click.native="go('forward')"
        ><svg-icon icon-class="arrow-right"
      /></button-icon>
    </div>
    <div class="navigation-links">
      <router-link to="/" :class="{ active: this.$route.name === 'home' }">{{
        $t("nav.home")
      }}</router-link>
      <router-link
        to="/explore"
        :class="{ active: this.$route.name === 'explore' }"
        >{{ $t("nav.explore") }}</router-link
      >
      <router-link
        to="/library"
        :class="{ active: this.$route.name === 'library' }"
        >{{ $t("nav.library") }}</router-link
      >
    </div>
    <div class="right-part">
      <a
        href="https://github.com/qier222/YesPlayMusic"
        target="blank"
        v-if="settings.showGithubIcon !== false"
        ><svg-icon icon-class="github" class="github"
      /></a>
      <button-icon @click.native="toggleSearchBox()" class="search-button">
        <svg-icon icon-class="search" />
      </button-icon>
      <div class="search-box">
        <div class="container" :class="{ active: inputFocus }">
          <svg-icon icon-class="search" />
          <div class="input">
            <input
              ref="searchInput"
              :placeholder="inputFocus ? '' : $t('nav.search')"
              v-model="keywords"
              @keydown.enter="doSearch"
              @focus="inputFocus = true"
              @blur="inputFocus = false"
            />
          </div>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
import ButtonIcon from "@/components/ButtonIcon.vue";
import { mapState } from "vuex";
const platformIsWin32 = window.require
  ? window.require("os").platform() == "win32"
    ? true
    : false
  : false;

const win = platformIsWin32
  ? window.require("electron").remote.getCurrentWindow()
  : null;

export default {
  name: "Navbar",
  components: {
    ButtonIcon,
  },
  data() {
    return {
      inputFocus: false,
      langs: ["zh-CN", "en"],
      keywords: "",
      isSearchBoxOpen: false,
      windowIsMaximized: win ? win.isMaximized() : true,
    };
  },
  computed: {
    ...mapState(["settings"]),
  },
  methods: {
    go(where) {
      if (where === "back") this.$router.go(-1);
      else this.$router.go(1);
    },
    doSearch() {
      if (!this.keywords) return;
      if (
        this.$route.name === "search" &&
        this.$route.params.keywords === this.keywords
      ) {
        return;
      }
      this.$router.push({
        name: "search",
        params: { keywords: this.keywords },
      });
    },
    toggleSearchBox() {
      this.isSearchBoxOpen = !this.isSearchBoxOpen;
    },
    windowMinimize() {
      win.minimize();
    },
    windowMaxRestore() {
      if (win.isMaximized()) {
        win.restore();
        this.windowIsMaximized = false;
      } else {
        win.maximize();
        this.windowIsMaximized = true;
      }
    },
    windowClose() {
      win.close();
    },
  },
};
</script>

<style lang="scss" scoped>
nav {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 64px;
  padding: var(--main-content-padding);
  width: calc(100vw - var(--main-content-padding-x) * 2);
  backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 1px solid transparent;
  transition-property: padding-bottom, border-bottom;
  transition-duration: 0.4s;

  background-color: var(--color-navbar-bg);
  z-index: 100;
  -webkit-app-region: drag;
}

@supports (-moz-appearance: none) {
  nav {
    background-color: var(--color-body-bg);
  }
}

.win32-titlebar {
  display: none;
}

[data-electron-platform-win32="yes"] {
  nav {
    padding-top: 20px;
    -webkit-app-region: no-drag;
  }
  .win32-titlebar {
    color: var(--color-text);
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    -webkit-app-region: drag;
    display: flex;
    align-items: center;
    --hover: #e6e6e6;
    --active: #cccccc;

    .title {
      padding: 8px;
      font-size: 12px;
      font-family: "Segoe UI", "Microsoft YaHei UI", "Microsoft YaHei",
        sans-serif;
    }
    .controls {
      height: 32px;
      margin-left: auto;
      justify-content: flex-end;
      display: flex;
      .button {
        height: 100%;
        width: 46px;
        font-size: 16px;
        display: flex;
        justify-content: center;
        align-items: center;
        -webkit-app-region: no-drag;
        &:hover {
          background: var(--hover);
        }
        &:active {
          background: var(--active);
        }
        &.close {
          &:hover {
            background: rgba(232, 17, 35, 0.9);
          }
          &:active {
            background: #f1707a;
            color: #000;
          }
        }
      }
    }
  }
  &[data-theme="dark"] .win32-titlebar {
    --hover: #191919;
    --active: #333333;
  }
}

.navigation-buttons {
  flex: 1;
  display: flex;
  align-items: center;
  .svg-icon {
    height: 24px;
    width: 24px;
  }
  button {
    -webkit-app-region: no-drag;
  }
}
@media (max-width: 970px) {
  .navigation-buttons {
    flex: unset;
  }
}

.navigation-links {
  flex: 1;
  display: flex;
  justify-content: center;
  text-transform: uppercase;
  user-select: none;
  a {
    -webkit-app-region: no-drag;
    font-size: 18px;
    font-weight: 700;
    text-decoration: none;
    border-radius: 6px;
    padding: 6px 10px;
    color: var(--color-text);
    transition: 0.2s;
    margin: {
      right: 12px;
      left: 12px;
    }
    &:hover {
      background: var(--color-secondary-bg-for-transparent);
    }
    &:active {
      transform: scale(0.92);
      transition: 0.2s;
    }
  }
  a.active {
    color: var(--color-primary);
  }
}

.search {
  .svg-icon {
    height: 18px;
    width: 18px;
  }
}

.search-box {
  display: flex;
  justify-content: flex-end;
  -webkit-app-region: no-drag;

  .container {
    display: flex;
    align-items: center;
    height: 32px;
    background: var(--color-secondary-bg-for-transparent);
    border-radius: 8px;
    width: 200px;
  }

  .svg-icon {
    height: 15px;
    width: 15px;
    color: var(--color-text);
    opacity: 0.28;
    margin: {
      left: 8px;
      right: 4px;
    }
  }

  input {
    font-size: 16px;
    border: none;
    background: transparent;
    width: 96%;
    font-weight: 600;
    margin-top: -1px;
    color: var(--color-text);
  }

  .active {
    background: var(--color-primary-bg-for-transparent);
    input,
    .svg-icon {
      opacity: 1;
      color: var(--color-primary);
    }
  }
}

[data-theme="dark"] {
  .search-box {
    .active {
      input,
      .svg-icon {
        color: var(--color-text);
      }
    }
  }
}

.right-part {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  .github {
    margin-right: 16px;
    height: 24px;
    width: 24px;
    color: var(--color-text);
    -webkit-app-region: no-drag;
  }
  @media (max-width: 400px) {
    .github {
      display: none;
    }
  }
  .search-button {
    display: none;
    -webkit-app-region: no-drag;
  }
  @media (max-width: 600px) {
    .search-button {
      display: flex;
    }
  }
}

@media (max-width: 600px) {
  .right-part {
    flex: 0;
  }
}

@media (max-width: 800px) {
  .navigation-links > a {
    margin: 4px;
  }
}

@media (max-width: 700px) {
  .navigation-buttons {
    display: none;
  }
}

@media (max-width: 600px) {
  nav.search-box-open {
    padding-bottom: 36px;
    border-bottom-color: var(--color-secondary-bg-for-transparent);
  }

  .search-box {
    display: block;
    position: fixed;
    top: 56px;
    left: 16px;
    right: 16px;
    .container {
      width: 100%;
      opacity: 0;
      height: 0;
      overflow: hidden;
      transition-property: height, opacity;
      transition-duration: 0.4s;
      .input {
        width: 100%;
      }
    }
  }

  [data-electron-platform-win32="yes"] {
    .search-box {
      // Add more 20px to top for title bar
      top: calc(56px + 20px);
    }
  }

  nav.search-box-open .container {
    opacity: 1;
    height: 32px;
  }
}
</style>
