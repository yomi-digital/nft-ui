<template>
  <div id="app">
    <div
      style="
        height: 50px;
        padding-top: 10px;
        border-bottom: 1px solid #fff;
        margin-bottom: 30px;
      "
    >
      NFT GENERATOR
    </div>
    <router-view />
    <hr />
    Developed with ❤️ by <a href="https://yomi.digital" target="_blank">YOMI</a>
  </div>
</template>

<style>
#app {
  font-family: "VT323", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  background: #121212;
  min-height: 100vh;
}
.label {
  color: #fff !important;
  text-align: left!important;
}
.title {
  color:#fff!important;
}
pre {
  border-radius: 10px;
  background: #444 !important;
  color: #fff !important;
}
.button {
  background: #444 !important;
  color: #fff !important;
  border-color: #fff !important;
}
.input,
textarea {
  background-color: transparent !important;
  border-bottom: 1px solid #fff;
  border-radius: 0px !important;
  border-top: 0 !important;
  border-left: 0 !important;
  border-right: 0 !important;
  text-align: left !important;
  color:#fff!important;
}
.textarea {
  border-left: 1px solid #fff!important;
  color:#fff!important;
  border-bottom: 0 !important;
  text-align: left !important;
}
</style>
