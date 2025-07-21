<template>
  <div class="container">
    <div class="input-group">
      <label>Ширина:</label>
      <input type="number" v-model.number="width" placeholder="100" />
    </div>
    <div class="input-group">
      <label>Высота:</label>
      <input type="number" v-model.number="height" placeholder="100" />
    </div>
    <button @click="randomColor">Случайный цвет</button>

    <div class="divider"></div>

    <div class="box"></div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      width: null,
      height: null,
      bgColor: '#d9edf7', 
      defaultSize: 170
    };
  },
  computed: {
    boxWidth() {
      return this.width || this.defaultSize;
    },
    boxHeight() {
      return this.height || this.defaultSize;
    }
  },
  methods: {
    randomColor() {
      const r = Math.floor(Math.random() * 256);
      const g = Math.floor(Math.random() * 256);
      const b = Math.floor(Math.random() * 256);
      this.bgColor = `rgb(${r}, ${g}, ${b})`;
    }
  }
};
</script>

<style scoped>
.container {
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #fff;
  max-width: 300px;
  margin: 0 auto;
}

.input-group {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

label {
  width: 70px;
  font-weight: 400;
}

input[type="number"] {
  width: 200px;
  padding: 5px; 
  height: 25px; 
  background-color: white;
  border: 1px solid black; 
  border-radius: 0;
  box-sizing: border-box;
}

input[type="number"]::placeholder {
  color: #c8c4c1;
}

button {
  width: 194px;
  padding: 1px;
  height: 25px; 
  margin-left: 66px;
  background-color: #e6e6e6; 
  color: black;
  border: 1px solid black; 
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.divider {
  width: 100%;
  height: 1px;
  background-color: #d6d6d6;
  margin: 15px 0;
}

.box {
  width: v-bind('boxWidth + "px"');
  height: v-bind('boxHeight + "px"');
  background-color: v-bind('bgColor');
  border: 2px solid black;
  border-radius: 15px;
  margin: 0 auto;
}
</style>
