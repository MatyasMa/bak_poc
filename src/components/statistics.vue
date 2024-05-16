<template>
  <div class="web">
    <div class="form_input_data">
      <form @submit.prevent="addData" method="post" >
        <h2>Saving shooting statistics</h2>
        <label for="x_value">X</label>
        <input type="number" name="x_value" id="x_value" max="10" min="-10" v-model="formData.x">
        <label for="y_value">Y</label>
        <input type="number" name="y_value" id="y_value" max="10" min="-10" v-model="formData.y">
        <label for="distance">Choose distance</label>
        <select name="distance" id="distance" v-model="formData.distance" required>
          <option value="50m">50m</option>
          <option value="100m">100m</option>
          <option value="150m">150m</option>
        </select>
        <button type="submit">
          Save
        </button>
        <button type="button" @click="deleteData">
          Delete all data
        </button>
      </form>
    </div>

    <div class="data">
      <table v-if="formRecords.length > 0">
        <tr>
          <th>X</th>
          <th>Y</th>
          <th>Distance</th>
        </tr>
        <tr v-for="record in formRecords" :key="record.x">
          <td>{{ record.x }}</td>
          <td>{{ record.y }}</td>
          <td>{{ record.distance }}</td>
        </tr>
      </table>
      <h2 v-else>Here will be displayed your data</h2>
    </div>
  </div>

  <div>
    <canvas id="myChart"></canvas>
  </div>
  

</template>

<script>
import Chart from 'chart.js/auto';

export default {
  data() {
    return {
      formData: {
        x: '',
        y: '',
        distance: ''
      },
      formRecords: JSON.parse(localStorage.getItem('formRecords')) || []
    };
  },
  mounted() {
    this.renderChartChartJS();
  },
  methods: {
    addData() {
      // Přidání dat formuláře do localStorage a aktualizace formRecords
      this.formRecords.push({ ...this.formData });
      localStorage.setItem('formRecords', JSON.stringify(this.formRecords));

      console.log('Data byla přidána.');
      console.log(this.formData);
      
      //this.formData = { x: '', y: '', distance: '' };
      // Resetování formuláře
      this.renderChartChartJS();
    },
    deleteData() {
      localStorage.clear();
      location.reload();
    },
    renderChartChartJS() {
      // Získání uložených dat
      const formRecords = JSON.parse(localStorage.getItem('formRecords')) || [];
      const place = document.getElementById('myChart').getContext('2d');
      const distances = Array.from(new Set(formRecords.map(record => record.distance))); // Unikátní vzdalenost

      const datasets = distances.map((distance, index) => {
        const dataByDistance = formRecords.filter(record => record.distance === distance);
        const color = this.getDistanceColor(index); // Získání barvy pro každou vzdalenost
        return {
          label: distance,
          data: dataByDistance.map(record => ({ x: record.x, y: record.y })),
          backgroundColor: color,
          borderColor: color,
          borderWidth: 1
        };
      });

      // Pokud již existuje instance grafu, zničíme ji
      if (this.myChart) {
        this.myChart.destroy();
      }

      // Tvorba grafu
      this.myChart = new Chart(place, {
        type: 'scatter',
        data: {
          datasets // Vlozena data ulozena podle vzdalenosti
        },
        options: {
          scales: {
            x: {
              min: -10,
              max: 10
            },
            y: {
              min: -10,
              max: 10, 
              beginAtZero: false
            }
          },
          plugins: {
            title: {
              display: true,
              text: 'Shooting statistics'
            },
            legend: {
              display: true,
              position: 'bottom'
            }
          }
        }
      });
    },
    getDistanceColor(index) {
      const colors = ['#ff6384', '#36a2eb', '#ffce56']; // Barvy pro každou kategorii
      return colors[index % colors.length]; // Pokud máte více kategorií, cyklus se opakuje
    }

  } 
};
</script>

<style>
* {
  color: black;
}
.web {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  column-gap: 50px;
  row-gap: 100px;
  margin-bottom: 50px;
  padding: 5%;
}
.web div {
  width: 45%;
}

/** form */
.form_input_data {
	min-width: 250px;
	width: 100%;
	display: flex;
	flex-wrap: wrap;
	justify-content: center;
	align-items: center;
	row-gap: 10px;
  background-color: whitesmoke;
  box-shadow: 10px 10px 10px lightgrey;
  border-radius: 30px;
  padding: 5%;
}
.form_input_data form {
  width: 90%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: row;
  flex-wrap: wrap;
  row-gap: 10px;
  column-gap: 10px;
}
.form_input_data form input[type=number] {
	width: 40%;
	padding: 10px;
  font-size: 1rem;
}
.form_input_data form label {
	width: 40%;
  color: black;
  text-align: center;
}
.form_input_data form select {
	width: 50%;
	padding: 10px;
  font-size: 1rem;
}
.form_input_data form button {
	width: 45%;
	padding: 10px;
  font-size: 1rem;
  margin-top: 20px;
  background-color: rgb(42, 173, 255);
  font-weight: bold;
  color: white;
  border: none;
  cursor: pointer;
  transition: 300ms ease all;
  border-radius: 10px;
}
.form_input_data form button:hover {
  background-color: rgb(4, 159, 255);
}

/** table */
.data table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.data th, .data td {
  padding: 8px;
  border: 1px solid #ddd;
  text-align: left;
}

.data th {
  background-color: #c8c8c8;
  text-align: center;
  border-bottom: 1px solid black;
  font-weight: bold;
}

.data tr:nth-child(even) {
  background-color: #f2f2f2;
}

.data tr:hover {
  background-color: #ddd;
}

.data h2 {
  margin-top: 20px;
}

</style>