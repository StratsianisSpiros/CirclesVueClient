<template>
  <div id="app">
    <div class="row">
      <form>
        <div class="mb-2 col-md-2 offset-3 form-group">
          <p style="color: #f47421;">Code</p>
          <p class="form-control" style="background-color: #e7f0f7;">{{circle.uniqueCode}}</p>
        </div>
        <div class="mb-2 col-md-2 offset-3 form-group">
          <label>X Coordinates </label>
          <input class="form-control" ref="coordX" type="number" v-model="circle.coordX" />
        </div>
        <div class="mb-2 col-md-2 offset-3 form-group">
          <label>Y Coordinates </label>
          <input class="form-control" ref="coordY" type="number" v-model="circle.coordY" />
        </div>
        <div class="mb-2 col-md3 col-md-2 offset-3 form-group">
          <a v-on:click="submitCircle()" class="btn btn-outline-primary">Draw Circle</a>
          <a v-on:click="show()" class="btn btn-outline-primary">{{!showAllCircles ? "Show Cirlces" : "Hide Circles"}}</a>
        </div>              
      </form>

      <div class="container">
        <canvas id="myCanvas" width="1000" height="500" v-on:click="getCursorPosition($event)"/>
      </div>
    </div>

  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",

  computed: {
    search: () => window.location.pathname.length < 2 ? "code" : window.location.pathname.replace("/", ""),
  },

  data() {
    return {
      circle: {
        uniqueCode: "",
        date: Date,
        coordX: 0,
        coordY: 0,
        hostUrl: window.location.host,
        diameter: Number,
        color: String,
      },
      circles: [],
      apiUrl: "https://localhost:44322/api/",
      request: String,
      showAllCircles: false
    };
  },

  mounted() {
    
    //checks with the database if code exist
    axios.get(this.apiUrl + 'code/' + this.search).then( response => {
      if (this.search === response.data) {
            this.getCirclesByCode(this.search)
      }

      this.circle.uniqueCode = response.data
      window.history.replaceState("", "", response.data)
    }, error => {
       console.log(error);
    });   
    
  },

  methods: {
    randomColor() { return "#" + Math.floor(Math.random() * 16777215).toString(16); },

    randomDiameter(value) { return Math.floor(Math.random() * value); },

    show() { 
      console.log(this.showAllCircles)
      this.showAllCircles = !this.showAllCircles;
      if (this.showAllCircles) {
        this.getCirclesByCode(this.circle.uniqueCode)
      }
      else {
        let canvas = document.getElementById("myCanvas");
        let ctx = canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }
    },

    //sets coordinates from user input with button
    setCoord(canvas) {
      let coordX = parseInt(this.$refs.coordX.value);
      if (coordX > canvas.width) {
        coordX = canvas.width;    
      }

      if (coordX < 0) {
        coordX = 0;
      }

      let coordY = parseInt(this.$refs.coordY.value);
      if (coordY > canvas.height) {
        coordY = canvas.height;
      }
            
      if (coordY < 0) {
        coordY = 0;
      }

      this.circle.coordY = coordY; 
      this.circle.coordX = coordX;
    },

    //Gets coordinates on canvas from mouse click
    getCursorPosition(event) {
      let canvas = document.getElementById("myCanvas");
      const rect = canvas.getBoundingClientRect()

      this.circle.coordX = event.clientX - rect.left
      this.circle.coordY = event.clientY - rect.top

      this.drawCircle();
    },

    //method to display circles on canvas
    showCircles() {
      let canvas = document.getElementById("myCanvas");
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      for (let c of this.circles) {
        ctx.beginPath();
        ctx.arc(c.coordX, c.coordY, c.diameter, 0, Math.PI * 2);
        ctx.fillStyle = c.color;
        ctx.fill();
      }
    },

    //method to draw circle o canvas with the current circle values
    drawCircle() {
      let canvas = document.getElementById("myCanvas");
      let ctx = canvas.getContext("2d");

      if (!this.showAllCircles) {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }

      ctx.beginPath();

      this.circle.date = new Date();
      this.circle.diameter = this.randomDiameter(canvas.height / 4);
      this.circle.color = this.randomColor();

      ctx.arc(this.circle.coordX, this.circle.coordY, this.circle.diameter, 0, Math.PI * 2);
      ctx.fillStyle = this.circle.color;
      ctx.fill();

      axios.post(`${this.apiUrl}store`, this.circle).then(response => {
        console.log(response);
      }, error => {
         console.log(error);
      })
    },

    submitCircle() {
      let canvas = document.getElementById("myCanvas");
      this.setCoord(canvas)
      this.drawCircle();
    },

    //Gets and draws all circles found in with the given unique code
    getCirclesByCode(code) {
      axios.get(this.apiUrl + 'circles/' + code).then((result) => {
        this.circles = result.data;
        this.showCircles();
      }, error => {
        console.log(error);
      });
    }
  },
  components: {},
};
</script>

<style>
@import url("../node_modules/bootstrap/dist/css/bootstrap.css");
@import url("../node_modules/bootstrap-vue/dist/bootstrap-vue.css");
@import url("App.css");
</style>
