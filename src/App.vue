<template>
  <div id="app">
    <h1>TODOS</h1>
    <router-view />
  </div>
</template>

<style lang="scss">
html,
body {
  overflow: hidden;
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}
ul,
li {
  margin: 0;
  padding: 0;
  list-style: none;
}
button {
  margin: 0;
  padding: 0;
  border: 0;
  background: none;
  font-size: 100%;
  vertical-align: baseline;
  color: inherit;
  outline: none;
  appearance: none;
}
input[type="checkbox"] {
  margin: 0;
  border: none;
  background-color: #fff;
  cursor: pointer;
  outline: none;
  appearance: none;
}
#app {
  overflow: auto;
  box-sizing: border-box;
  padding: 10px;
  height: 100%;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  text-align: center;
  color: #ff8c97;
  background: #f5f5f5;

  h1 {
    margin: 0;
    padding: 30px;
    font-size: 4em;
    font-weight: bold;
  }
}
</style>
