<template>
  <div class="wallet-container">
    <div class="welcome-container" v-show="!loading">
      <div class="logo-container">
        <img src="../static/main-logo.png" width="136" />
        <!-- <div class="slogin">{{ $t('common.slogin') }}</div> -->
      </div>

      <div class="login-container" v-show="vaultCreated && isLocked">
        <div class="password-wrap">
          <input
            class="display-block"
            :class="{error: error}"
            :type="type"
            v-model="password"
            @focus="focusHandler"
            @blur="blurHandler"
            @input="inputHandler"
            ref="password1"
          />
          <div class="holder" :class="{shouldTop: shouldTop, deep: error}">{{$t('wallet.import.password')}}</div>
          <div class="error-msg" v-if="error">{{ error }}</div>
          <div class="actions" v-if="showClear">
            <div class="action clear" @click="setClear"></div>
            <div class="action line"></div>
            <div
              class="action check"
              :class="{cansee: cansee}"
              v-if="showClear"
              @click="toggleCansee"
            ></div>
          </div>
        </div>
        <button
          class="display-block btn-primary"
          :disabled="password.length < 6 || password.length > 20"
          @click="unlock"
        >{{ $t('splash.unlockButton') }}</button>
        <div class="recover" @click="importWallet">{{ $t('splash.restoreWalletButton') }}</div>
      </div>

      <div class="create-container" v-show="!vaultCreated">
        <button
          class="display-block btn-primary btn-higher"
          @click="createWallet"
        >{{ $t('splash.createWalletButton') }}</button>
        <button
          class="display-block btn-higher btn-lighter"
          @click="importWallet"
        >{{ $t('splash.importWalletButton') }}</button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.welcome-container {
  position: relative;
  height: 100%;
  box-sizing: border-box;
}

.logo-container {
  padding-top: 90px;
  text-align: center;
  .slogin {
    margin-top: 50px;
    color: #7c8bb9;
    font-size: 17px;
    padding: 0 30px;
    line-height: 28px;
    font-weight: 300;
  }
}

.create-container {
  position: absolute;
  left: 16px;
  right: 16px;
  bottom: 48px;
  padding: 0 60px;
}
.login-container {
  position: absolute;
  left: 24px;
  right: 24px;
  bottom: 44px;
  .recover {
    text-align: center;
    color: #062deb;
    font-size: 14px;
    line-height: 22px;
    margin-top: 16px;
    cursor: pointer;
    &:hover {
      text-decoration: underline;
    }
  }
  .password-wrap {
    position: relative;
    height: 50px;
    margin-bottom: 40px;
    .error-msg {
      white-space: nowrap;
      line-height: 52px;
      position: absolute;
      z-index: 10;
      left: 0;
      bottom: -59px;
      font-size: 24px;
      padding: 0 16px;
      color: #ed4b4d;
      background-color: #ffe6e5;
      border-radius: 6px;
      z-index: 10;
      border: 1px solid #e74b4d;
      transform: scale(0.5);
      transform-origin: left top;
      &:after {
        content: "";
        position: absolute;
        left: 20px;
        top: -15%;
        width: 14px;
        height: 14px;
        border-left: 1px solid rgb(226, 66, 68);
        border-top: 1px solid rgb(226, 66, 68);
        background-color: #ffe6e5;
        transform: rotate(45deg);
        z-index: 0;
        box-sizing: border-box;
        border-top-left-radius: 4px;
      }
    }
    .holder {
      position: absolute;
      left: 14px;
      top: 15px;
      font-size: 15px;
      color: #8187a5;
      line-height: 20px;
      background-color: #fff;
      z-index: 1;
      opacity: 0.7;
      transition: all 150ms linear;
      &.deep {
        color:  #f75555;
      }
      &.shouldTop {
        padding: 0 6px;
        top: -10px;
        z-index: 3;
        font-size: 12px;
        opacity: 1;
      }
    }
    input {
      position: absolute;
      left: 0;
      top: 0;
      margin: 0;
      width: 100%;
      height: 50px;
      z-index: 2;
      background: none;
      padding-left: 14px;
      padding-right: 83px;
      &:focus {
        border-color: #8187a5;
      }
      &.error {
        border-color: #f75555;
      }
    }
  }
  .actions {
    position: absolute;
    right: 0px;
    top: 0px;
    z-index: 3;
    height: 50px;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    .action {
      background-color: #ccc;
      margin-right: 12px;
      cursor: pointer;
    }
    .clear {
      margin-left: 12px;
      width: 16px;
      height: 16px;
      background: url("../static/input-clear.svg");
    }
    .line {
      width: 1px;
      height: 17px;
      background-color: #d1d2da;
    }
    .check {
      width: 18px;
      height: 14px;
      background: url("../static/cansee.svg");
      &.cansee {
        background: url("../static/cannotsee.svg");
      }
    }
  }
}
</style>

<script type="text/jsx">
import API from "../api";
const ver = require("../../../package.json").version;
let _ = require("lodash");

export default {
  data() {
    return {
      loading: true,
      isLocked: true,
      vaultCreated: false,
      password: "",
      shouldTop: false,
      showClear: false,
      cansee: false,
      type: "password",
      error: ""
    };
  },

  beforeCreate() {
    sessionStorage.removeItem("password");
    sessionStorage.removeItem("password2");
    sessionStorage.removeItem("isAgreed");
  },

  created() {
    API.getState().then(
      state => {
        if (!sessionStorage.getItem("isActive")) {
          API.getExtensionVersion(state.network, {
            appNo: 1,
            langType: 1,
            packageType: 1,
            platformType: "360safe",
            version: ver
          }).then(res => {
            if (+res.upGradeType) {
              this.$router.push({
                path: "/wallet/update",
                query: {
                  content:
                    this.$i18n.locale.indexOf("zh") > -1
                      ? res.releaseNoteZh
                      : res.releaseNoteEn
                }
              });
            }
          });
        }
        this.loading = false;
        this.isLocked = state.isLocked;
        this.vaultCreated = state.vaultCreated;

        if (!this.isLocked || !this.vaultCreated) {
          const restoreConfigString = localStorage.getItem("WICC_RESTORE_PATH");

          if (restoreConfigString) {
            try {
              const restoreConfig = JSON.parse(restoreConfigString);
              if (restoreConfig) {
                return this.$router.push(restoreConfig);
              }
            } catch (error) {
              console.log("parse restore config error:", error);
            } finally {
              localStorage.setItem("WICC_RESTORE_PATH", null);
            }
          }

          if (this.vaultCreated) {
            this.gotoMain();
          }
        }
      },
      error => {
        this.loading = false;
        //    throw error
      }
    );
    setTimeout(() => {
      if (!this.$refs.password1) return;
      this.$refs.password1.onkeyup = data => {
        if (data.keyCode === 13) {
          this.unlock();
        }
      };
    }, 200);
  },

  beforeDestroy() {
    this.$refs.password1.onkeyup = null;
  },

  watch: {
    password(val) {
      if (val) {
        this.showClear = true;
      } else {
        this.showClear = false;
      }
    },
    cansee(val) {
      if (val) {
        this.type = "text";
      } else {
        this.type = "password";
      }
    }
  },

  methods: {
    inputHandler() {
      this.error = "";
      this.password = this.password.replace(/\s+/gi, item => "");
    },
    focusHandler() {
      this.shouldTop = true;
    },
    blurHandler() {
      if (this.password) return;
      this.shouldTop = false;
    },
    setFocus() {
      this.$refs.password1 && this.$refs.password1.focus();
    },
    setClear() {
      this.password = "";
      this.error = "";
      this.setFocus();
      setTimeout(() => {
        this.showClear = false;
      }, 10);
    },
    toggleCansee() {
      this.cansee = !this.cansee;
    },
    gotoMain() {
      this.$router.push({
        name: "accountMain"
      });
    },

    unlock() {
      API.unlock(this.password).then(
        () => {
          this.gotoMain();
        },
        error => {
          // this.$toast(this.$t("errors.passwordError"), {
          //   type: "center"
          // });
          this.error = this.$t("errors.passwordError");
          // throw error
        }
      );
    },

    createWallet() {
      this.checkNetWork();
      this.$router.push({
        name: "createWallet"
      });
    },

    importWallet() {
      this.checkNetWork();
      this.$router.push({
        name: "importWallet"
      });
    },
    checkNetWork() {
      const localNet = localStorage.getItem("network");
      if (localNet) {
        return;
      }
      localStorage.setItem("network", "mainnet");
    }
  }
};
</script>
