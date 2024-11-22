<template>
  <div>
    <Header :toggleMenu="toggleMenu" />
    <div class="statistic-chart">
      <h1>Post Statistics</h1>

      <!-- Date Filter -->
      <div class="date-filter">
        <label for="start-date">Start Date:</label>
        <input type="date" id="start-date" v-model="startDate" />

        <label for="end-date">End Date:</label>
        <input type="date" id="end-date" v-model="endDate" />

        <button @click="fetchStatistics">Update</button>
      </div>

      <!-- Chart -->
      <div class="chart-container">
        <canvas id="post-chart"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Chart from "chart.js/auto";
import Header from "~/components/Header.vue";

export default {
  components: {
    Header,
  },
  data() {
    return {
      userId: null, // User ID
      startDate: "", // Start date
      endDate: "", // End date
      chart: null, // Chart object
    };
  },
  methods: {
    async fetchStatistics() {
      if (!this.startDate || !this.endDate) {
        alert("Please select both dates.");
        return;
      }

      if (!this.userId) {
        alert("User ID is not available.");
        return;
      }

      try {
        // Fetch data from API
        const response = await axios.get(
          `http://127.0.0.1:8000/api/users/${this.userId}/get_statistic`,
          {
            params: {
              start_date: this.startDate,
              end_date: this.endDate,
            },
          }
        );

        // Build chart with data
        this.updateChart(response.data);
      } catch (error) {
        console.error("Error fetching statistics:", error);
        alert("Failed to load statistics.");
      }
    },
    updateChart(data) {
      const labels = data.map((item) => item.date); // Dates for X-axis
      const counts = data.map((item) => item.count); // Post counts for Y-axis

      // Destroy old chart if exists
      if (this.chart) {
        this.chart.destroy();
      }

      // Create new chart
      const ctx = document.getElementById("post-chart").getContext("2d");
      this.chart = new Chart(ctx, {
        type: "bar", // Chart type
        data: {
          labels: labels,
          datasets: [
            {
              label: "Number of Posts",
              data: counts,
              backgroundColor: "rgba(54, 162, 235, 0.5)", // Bar color
              borderColor: "rgba(54, 162, 235, 1)", // Border color
              borderWidth: 1, // Border width
            },
          ],
        },
        options: {
          responsive: true,
          scales: {
            x: {
              title: {
                display: true,
                text: "Date", // X-axis title
              },
            },
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: "Number of Posts", // Y-axis title
              },
              ticks: {
                stepSize: 1, // Шаг между метками
                callback: function (value) {
                  if (Number.isInteger(value)) {
                    return value; // Возвращаем только целые значения
                  }
                  return null; // Пропускаем дробные значения
                },
              },
            },
          },
        },
      });
    },
  },
  mounted() {
    // Get userId from localStorage
    const storedUserId = localStorage.getItem("userId");
    if (storedUserId) {
      this.userId = parseInt(storedUserId, 10); // Convert to integer
    } else {
      console.warn("User ID is not found in localStorage.");
    }

    // Set default values for filter
    const today = new Date().toISOString().split("T")[0];
    this.startDate = "2024-01-01"; // Default start date
    this.endDate = today;

    // Load data on initial render if userId exists
    if (this.userId) {
      this.fetchStatistics();
    }
  },
};
</script>

<style>
/* General styles for the page */

/* Statistics Card */
.statistic-chart {
  background: #ffffff;
  color: #333;
  border-radius: 20px;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  padding: 20px;
  max-width: 800px;
  width: 100%;
  margin: 10px auto;
}

/* Title */
.statistic-chart h1 {
  font-size: 24px;
  text-align: center;
  margin-bottom: 20px;
}

/* Date Filter */
.date-filter {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
  align-items: center;
}

.date-filter label {
  font-weight: bold;
  font-size: 16px;
}

.date-filter input {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 14px;
  width: 100%;
  max-width: 250px;
}

/* Update Button */
.date-filter button {
  padding: 10px 20px;
  background-color: #6a11cb;
  color: #ffffff;
  font-size: 16px;
  font-weight: bold;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.date-filter button:hover {
  background-color: #2575fc;
}

/* Chart */
.chart-container {
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 20px;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
}

canvas {
  width: 100%;
  height: 300px;
}
</style>
