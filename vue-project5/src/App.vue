<template>
  <div class="table-container">
    <div class="spreadsheet-wrapper">
      <table>
        <tbody>
          <tr v-for="(row, rowIndex) in table" :key="rowIndex">
            <td v-for="(cell, colIndex) in row" :key="colIndex" @dblclick="editCell(rowIndex, colIndex)">
              <input v-if="editingCell.row === rowIndex && editingCell.col === colIndex"
                     v-model="table[rowIndex][colIndex]"
                     @blur="saveCell"
                     @keyup.enter="saveCell"
                     type="text"
                     class="cell-input"
                     autofocus />
              <span v-else>{{ cell }}</span>
            </td>
          </tr>
        </tbody>
      </table>

      <div class="column-buttons">
        <button @click="addColumn">+</button>
        <button @click="removeColumn">-</button>
      </div>
    </div>

    <div class="row-buttons">
      <button @click="addRow">+</button>
      <button @click="removeRow">-</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Spreadsheet',
  data() {
    return {
      table: [],
      editingCell: { row: null, col: null },
      minRows: 1,
      minCols: 1
    };
  },
  mounted() {
    const saved = JSON.parse(localStorage.getItem('spreadsheet'));
    if (saved && Array.isArray(saved) && saved.length > 0) {
      this.table = saved;
    } else {
      this.initTable(5, 5);
    }
  },
  methods: {
    initTable(rows, cols) {
      this.table = Array.from({ length: rows }, () => Array(cols).fill(''));
      this.saveToStorage();
    },
    editCell(row, col) {
      this.editingCell = { row, col };
    },
    saveCell() {
      this.editingCell = { row: null, col: null };
      this.saveToStorage();
    },
    addRow() {
      const cols = this.table[0]?.length || 0;
      this.table.push(Array(cols).fill(''));
      this.saveToStorage();
    },
    removeRow() {
      if (this.table.length <= this.minRows) return;
      const lastRow = this.table[this.table.length - 1];
      if (lastRow.some(cell => cell)) {
        if (!confirm("Удалить строку с данными?")) return;
      }
      this.table.pop();
      this.saveToStorage();
    },
    addColumn() {
      this.table.forEach(row => row.push(''));
      this.saveToStorage();
    },
    removeColumn() {
      if (this.table[0].length <= this.minCols) return;
      const colIndex = this.table[0].length - 1;
      const hasData = this.table.some(row => row[colIndex]);
      if (hasData && !confirm("Удалить столбец с данными?")) return;
      this.table = this.table.map(row => {
        row.pop();
        return row;
      });
      this.saveToStorage();
    },
    saveToStorage() {
      localStorage.setItem('spreadsheet', JSON.stringify(this.table));
    }
  }
};
</script>

<style scoped>
.table-container {
  font-family: sans-serif;
  padding: 20px;
}

table {
  border-collapse: collapse;
}

td {
  border: 1px solid #000000;
  padding: 1px;
  min-width: 80px;
  height: 30px;
  text-align: center;
  position: relative;
}

.cell-input {
  position: absolute;
  left: 0;
  top: 0;
  width: 98%;
  height: 94%;
  margin: 1px;
  padding: 1px;
  border: 1px solid #000000;
  outline: none;
  text-align: center;
  box-sizing: border-box;
  font-size: inherit;
  font-family: inherit;
}

button {
  margin: 4px;
  padding: 3px 10px;
  font-size: 16px;
  width: 35px;
  height: 25px;
  box-sizing: border-box;
}

.spreadsheet-wrapper {
  display: flex;
  align-items: flex-start;
}

.column-buttons {
  display: flex;
  flex-direction: column;
  margin-left: 10px;
}

.row-buttons {
  margin-top: 10px;
}
</style>