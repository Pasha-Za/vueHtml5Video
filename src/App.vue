<template>
  <div id="app">
    <div class="container">
      <videoplayer
        :source="[{ src: 'http://media.w3.org/2010/05/bunny/movie.mp4', type: 'mp4'}, { src: 'http://media.w3.org/2010/05/bunny/movie.mp4', type: 'mp4'}]"
        poster='http://i.vimeocdn.com/video/20963649_1280x720.jpg'
        :width='600'
        :muted='true'
        :autoplay='false'
        :controls='true'
      />
    </div>
  </div>
</template>

<script>
import Videoplayer from './components/Videoplayer'

export default {
  name: 'app',
  components: {
    Videoplayer
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.container{
  max-width: 1200px;
  margin: 0 auto;
}
</style>
