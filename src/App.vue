<script setup>
</script>

<template>
    <div class="container">
        <div class="form">
            <h3>Confira o clima de uma cidade:</h3>
            <div class="form-input-container">
                <input type="text" placeholder="Digite o nome da cidade" v-model="cityName" v-on:keyup.enter="searchWeather" />
                <button @click="searchWeather">Pesquisar</button> <!-- Botão de pesquisa adicionado -->
            </div>
        </div>
        <div v-if="weatherData" id="weather-data">
            <h2>
                <i class="fa-solid fa-location-dot"></i> {{ weatherData.city }} <img :src="weatherData.countryFlag"
                    alt="Bandeira do país" />
            </h2>
            <p id="temperature">{{ weatherData.temperature }}°C</p>
            <div id="description-container">
                <p id="description">{{ weatherData.description }}</p>
                <img id="weather-icon" :src="weatherData.iconUrl" alt="Condições atuais" />
            </div>
            <div id="details-container">
                <p id="umidity">
                    <i class="fa-solid fa-droplet"></i> {{ weatherData.humidity }}
                </p>
                <p id="wind">
                    <i class="fa-solid fa-wind"></i> {{ weatherData.wind }}
                </p>
            </div>
        </div>
        <div v-else id="error-message">
            <p>Não foi possível encontrar o clima de uma cidade com este nome.</p>
        </div>
        <div v-if="loading" id="loader">
            <i class="fa-solid fa-spinner"></i>
        </div>
        <div id="suggestions">
            <button v-for="city in suggestedCities" :key="city" @click="searchSuggestedCity(city)">{{ city }}</button>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            apiKey: "ba605efc18f1572f61892fe426f18a1a",
            apiCountryURL: "https://flagsapi.com",
            apiUnsplash: "https://source.unsplash.com/1600x900/?",
            cityName: "",
            weatherData: null,
            loading: false,
            suggestedCities: ["Manaus", "Hong Kong", "Zurique", "Vancouver", "Genebra", "Frankfurt", "Tokio", "Nova York"],
        };
    },
    methods: {
        showLoader() {
            this.loading = true;
        },
        hideLoader() {
            this.loading = false;
        },
        async getWeatherData(city) {
            this.showLoader();

            const apiWeatherURL = `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${this.apiKey}&lang=pt_br`;

            try {
                const res = await fetch(apiWeatherURL);
                const data = await res.json();
                console.log('Response:', res);
                console.log('Data:', data);

                if (!res.ok) {
                    throw new Error(`HTTP error! status: ${res.status}`);
                }

                return data;
            } catch (error) {
                console.error(`Fetch failed: ${error.message}`);
            }
            finally{
                this.hideLoader();
            }
        },
        async showWeatherData(city) {
            const data = await this.getWeatherData(city);

            if (data.cod === "404") {
                this.weatherData = null;
                return;
            }

            this.weatherData = {
                city: data.name,
                temperature: parseInt(data.main.temp),
                description: data.weather[0].description,
                iconUrl: `http://openweathermap.org/img/wn/${data.weather[0].icon}.png`,
                countryFlag: `https://www.flagsapi.com/${data.sys.country}/flat/64.png`,
                humidity: `${data.main.humidity}%`,
                wind: `${data.wind.speed}km/h`,
            };

            // Change bg image
            document.body.style.backgroundImage = `url("${this.apiUnsplash + city}")`;
        },
        async searchWeather() {
            this.showWeatherData(this.cityName);
        },
        async searchSuggestedCity(city) {
            this.cityName = city;
            this.searchWeather();
        },
    },
};
</script>