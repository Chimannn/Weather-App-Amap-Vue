<template>
  <v-app>
   <div id="app">
      <Header />
      <Weather />  
   </div>
  </v-app>
</template>

<script>
import Header from './components/Header.vue';
import Weather from './components/Weather.vue';

export default {
  name: 'App',

  components: {
    Header,
    Weather
  }
 
    

};
</script>

<style scoped>
  #app{
    background-image: url('./assets/img/rainbow.jpg');
    background-repeat: no-repeat;
    background-attachment: fixed;
    background-size: contain;
    background-position: center;
    text-align: center;
    /* animation: change 15s infinite; */
  }


  @keyframes change{
    0% {
      background-image: url('./assets/img/rainbow.jpg');
    }
    25% {
      background-image: url('./assets/img/rain.jpg');
    }
    50% {
      background-image: url('./assets/img/thunderstorm.jpg');
    }
    75% {
      background-image: url('./assets/img/background1.jpg');
    }
    100% {
      background-image: url('./assets/img/rainbow.jpg');
    }
  }

</style>
