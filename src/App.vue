<template>
  <main id="app">
    <header class="ribbon">
      <span class="ribbon-inner">
        <img class="logo" alt="DARC" src="./assets/darc.png" />
        <img class="logo" alt="TMC" src="./assets/Medical-City-logo.jpg" />
        <img class="logo" alt="DG" src="./assets/DG-resized.png" />
      </span>
    </header>
    <router-view />
  </main>
</template>

<style lang="scss">
@import "./scss/styles.scss";
#app {
  width: 100vw;
  max-width: 1200px;
  min-height: calc(100vh - 1.5rem);
  margin: 0 auto;
  position: relative;

  color: #333;
}

.ribbon {
  position: absolute;
  top:0;
  left: 0;
  background-color: $ribbon-background-color;
  color: $blue;
  display: inline-block;
  width: $ribbon-tri-width;
  text-decoration: none;
  padding: 0 #{$ribbon-tri-padding} #{$ribbon-tri-padding / 2};
  text-align: center;

  box-shadow: 2px 2px 3px rgba(0,0,0,0.2);

  &:after {
    content: '';
    display: block;
    width:0;
    height:0;
    border-style: solid;
    border-color: $ribbon-background-color transparent transparent transparent;
    border-width: #{$ribbon-tri-height - ($ribbon-tri-padding / 2) } #{$ribbon-tri-width / 2} 0 #{$ribbon-tri-width / 2};

    position: absolute;
    top: 100%;
    left: 0;
  }
}

.ribbon-inner {
    display: block;
    border-left: 1px solid $blue;
    border-right: 1px solid $blue;
    position: relative;
    height: 100%;
    padding: 2rem 1rem;

    &:before,
    &:after {
        content: '';
        display: block;

        //css triangle
        width:0;
        height:0;
        border-style: solid;
        border-color: $ribbon-background-color transparent transparent transparent;
        border-width: #{$ribbon-tri-height -$ribbon-tri-padding} #{($ribbon-tri-width / 2) - $ribbon-tri-padding} 0 #{($ribbon-tri-width / 2) - $ribbon-tri-padding};
        margin-top: -1px;

        position: absolute;
        top: 100%;
        left: -1px;
        z-index: 5;
    }


    &:before {
        border-color: $blue transparent transparent transparent;
        border-width: #{$ribbon-tri-height -$ribbon-tri-padding} #{($ribbon-tri-width / 2) - $ribbon-tri-padding} 0 #{($ribbon-tri-width / 2) - $ribbon-tri-padding};
        margin-top: 0;
    }
}

.logo {
  width: 100%;
  height: auto;
  margin: 0.3rem auto;

  &:first-of-type {
    margin-bottom: 0.5rem;
  }
}
</style>
