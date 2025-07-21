<template>
  <div class="app">
    <div class="filters">
      <label>Цена от: 
        <input type="number" 
          v-model.number="minPrice"
          placeholder="0" />
      </label>
      <label>до: 
        <input type="number" 
          v-model.number="maxPrice"
          placeholder="10000" />
      </label>
      <button @click="applyFilter">Обновить</button>
    </div>
    <div class="table-conteiner">
    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="filteredProducts.length > 0">
      <table class="product-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Название</th>
            <th>Количество</th>
            <th>Цена за единицу</th>
            <th>Сумма</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in filteredProducts" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ product.name }}</td>
            <td>{{ product.quantity }}</td>
            <td>{{ product.price }}</td>
            <td>{{ product.quantity * product.price }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-else-if="!error">Нет данных, попадающих под условие фильтра</div>
  </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      products: [],
      filteredProducts: [],
      minPrice: null,
      maxPrice: null,
      error: ''
    };
  },
  methods: {
    async fetchProducts() {
      try {
        const response = await fetch('/api/service/products/');
        if (!response.ok) {
          throw new Error(`Ошибка загрузки: ${response.status}`);
        }
        const data = await response.json();
        this.products = data;
        this.filteredProducts = data;
      } catch (err) {
        this.error = 'Не удалось загрузить данные.';
        console.error(err);
      }
    },
    applyFilter() {
      this.error = '';
      if (this.minPrice < 0 || this.maxPrice < 0) {
        this.error = 'Цены не могут быть отрицательными';
        return;
      }
      if (this.maxPrice !== 0 && this.minPrice > this.maxPrice) {
        this.error = 'Минимальная цена не может быть больше максимальной';
        return;
      }

      this.filteredProducts = this.products.filter((product) => {
        if (this.minPrice === 0 && this.maxPrice === 0) return true;
        if (this.maxPrice === 0) return product.price >= this.minPrice;
        return product.price >= this.minPrice && product.price <= this.maxPrice;
      });
  }
  },
  mounted() {
    this.fetchProducts();
  }
};
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
   min-height: 100vh;
  padding: 40px;

}

.filters {
  text-align: center;
  justify-content: center;
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  align-items: center;
}

.table-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 200px;
  border: 1px solid #ddd;
  padding: 10px;
  margin-top: 10px;
  min-width: 200px;
}

.filters input {
  width: 80px;
  height: 27px;
  padding: 5px;
  border: 2px solid #black;
}

.filters input::placeholder {
  color: #c8cdd9;
}

button {
  padding: 4px 12px;
  background-color: #e6e6e6;
  color: #black;
  border: 1px solid #black;
  cursor: pointer;
  border-radius: 5px;
  width: 120px;
  height: 27px;
  font-size: 16px;
}

.product-table {
  width: 130%;
  border-collapse: collapse;
  border: 1px solid #000000;
}

.product-table th,
.product-table td {
  padding: 10px;
  border-bottom: 1px solid #000000;
  text-align: left;
}

.product-table th {
  position: sticky;
  top: 20px;
  background-color: #cccccc;
}

.product-table th:not(:last-child),
.product-table td:not(:last-child) {
  border-right: none;
}

.error {
  color: red;
  margin-bottom: 15px;
}
</style>