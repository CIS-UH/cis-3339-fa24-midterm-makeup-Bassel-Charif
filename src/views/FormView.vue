<!--I asked chat gpt "How can i change this options api to composition api using the script setup function"-->
<template>
    <div>
        <form @submit.prevent="fetch_remote_data">
            <div class="form-group">
                <h2> Weather Data </h2>
                <p> Please fill in the form and use the submit button to load data.</p>
                <label for="">Location</label>
                <br>
                <input class="form-control rounded" v-model="form_input.location" type="text" required />
            </div>
            
            <div class="form-group">
                <label for="">Start Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="form_input.start_dt" required />
            </div>
            
            <div class="form-group">
                <label for="">End Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="form_input.end_dt" required />
            </div>
            <br>
            <div>
                <button class="btn btn-danger" type="submit">Submit</button>
            </div>
        </form>
        <br /><br />

        <div v-if="dataLoaded">
            <Bar :data="chart_data" :options="chart_options" />
        </div>

        <div v-if="dataLoaded">
            <hr>
            <h5> Weather Data in a Table</h5>
            <table class="table">
                <thead>
                    <tr>
                        <th v-for="adate in keys" :key="adate">{{ adate }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td v-for="atemp in values" :key="atemp">{{ atemp }}</td>
                    </tr>
                </tbody>
            </table>            
        </div>
    </div>
</template>

<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
</style>

<script setup>
import { ref, reactive } from 'vue'
import axios from "axios"
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
} from 'chart.js'
import { Bar } from 'vue-chartjs'

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)

// Form input data
const form_input = reactive({
    start_dt: "",
    end_dt: "",
    location: ""
})

// Chart options and data
const chart_options = reactive({
    responsive: true,
    maintainAspectRatio: true,
    scales: {
        x: {
            title: {
                display: true,
                text: "Time"
            }
        },
        y: {
            title: {
                display: true,
                text: "Temperature (F)"
            }
        }
    }
})

const chart_data = reactive({
    labels: [],
    datasets: [
        {
            label: "Average Temperature",
            backgroundColor: "#f87979",
            data: []
        }
    ]
})

// Other state variables
const dataLoaded = ref(false)
const keys = ref([])
const values = ref([])

// Fetch weather data from API
async function fetch_remote_data() {
    const options = {
        method: 'GET',
        url: 'https://weatherapi-com.p.rapidapi.com/history.json',
        params: {
            q: form_input.location,
            dt: form_input.start_dt,
            end_dt: form_input.end_dt,
            lang: 'en'
        },
        headers: {
            'X-RapidAPI-Key': '6c6f790650msh5ce4da2fced77a7p1b1776jsn247d9f531b74',
            'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
        }
    }
    
    try {
        const response = await axios.request(options)
        const w_data = response.data.forecast.forecastday
        keys.value = w_data.map((item) => item.date)
        values.value = w_data.map((item) => item.day.avgtemp_f)

        chart_data.labels = keys.value
        chart_data.datasets[0].data = values.value

        dataLoaded.value = true
    } catch (error) {
        console.error(error)
    }
}
</script>

