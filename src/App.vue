<template>
  <div id="app" :class="backgroundClass">
    <main>
      <div class="search-box">
        <input
          type="text"
          class="search-bar"
          placeholder="Enter city name..."
          v-model="query"
          @keypress.enter="fetchWeather"
        />
      </div>

      <div class="weather-wrap" v-if="weather.main">
        <div class="location-box">
          <div class="location">
            {{ weather.name }}, {{ weather.sys.country }}
          </div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <!-- Weather Icon -->
          <div class="data-cube">
            <img
              v-if="weather.weather[0].icon"
              :src="`http://openweathermap.org/img/wn/${weather.weather[0].icon}.png`"
              :alt="weather.weather[0].description"
              class="weather-icon"
            />
          </div>
          <!-- Weather Data Cubes -->
          <div class="data-row">
            <div class="data-cube">
              <div class="data-title">Temperature</div>
              <div class="data-value">
                {{ Math.round(weather.main.temp) }}°C
              </div>
            </div>
            <div class="data-cube">
              <div class="data-title">Weather</div>
              <div class="data-value">{{ weather.weather[0].main }}</div>
            </div>
            <div class="data-cube">
              <div class="data-title">Description</div>
              <div class="data-value">{{ weather.weather[0].description }}</div>
            </div>
            <div class="data-cube">
              <div class="data-title">Humidity</div>
              <div class="data-value">{{ weather.main.humidity }}%</div>
            </div>
            <div class="data-cube">
              <div class="data-title">Wind Speed</div>
              <div class="data-value">
                {{ Math.round(weather.wind.speed) }} m/s
              </div>
            </div>
            <div class="data-cube">
              <div class="data-title">Pressure</div>
              <div class="data-value">{{ weather.main.pressure }} hPa</div>
            </div>
            <div class="data-cube">
              <div class="data-title">Sunrise</div>
              <div class="data-value">{{ sunriseTime }}</div>
            </div>
            <div class="data-cube">
              <div class="data-title">Sunset</div>
              <div class="data-value">{{ sunsetTime }}</div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      api_key: "f1274ab0438bf206fe2b50262a57513e",
      url_base: "https://api.openweathermap.org/data/2.5/",
      query: "",
      weather: {},
      defaultLocation: { lat: -33.9249, lon: 18.4241 }, // Cape Town coordinates
      sunriseTime: "",
      sunsetTime: "",
      backgroundClass: "",
    };
  },
  mounted() {
    this.fetchWeatherForLocation(
      this.defaultLocation.lat,
      this.defaultLocation.lon
    );
  },
  methods: {
    fetchWeather(e) {
      if (e.key === "Enter" && this.query) {
        fetch(
          `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`
        )
          .then((res) => res.json())
          .then(this.setResults)
          .catch((error) => console.error("Error fetching weather:", error));
      }
    },
    fetchWeatherForLocation(lat, lon) {
      fetch(
        `${this.url_base}weather?lat=${lat}&lon=${lon}&units=metric&APPID=${this.api_key}`
      )
        .then((res) => res.json())
        .then(this.setResults)
        .catch((error) =>
          console.error("Error fetching default location weather:", error)
        );
    },
    setResults(results) {
      if (results.main) {
        this.weather = results;
        if (results.sys) {
          this.sunriseTime = this.formatTime(results.sys.sunrise);
          this.sunsetTime = this.formatTime(results.sys.sunset);
          this.updateBackground();
        }
      }
    },
    updateBackground() {
      const now = new Date();
      const sunrise = new Date(this.weather.sys.sunrise * 1000);
      const sunset = new Date(this.weather.sys.sunset * 1000);

      if (now >= sunrise && now <= sunset) {
        this.backgroundClass = "warm";
      } else {
        this.backgroundClass = "cold";
      }
    },
    dateBuilder() {
      let d = new Date();
      let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day}, ${date} ${month} ${year}`;
    },
    formatTime(timestamp) {
      let date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], {
        hour: "2-digit",
        minute: "2-digit",
      });
    },
  },
};
</script>

<style>
/* Add your CSS styles here */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: "Montserrat", sans-serif;
  background-color: #f4f4f9;
}
#app {
  background-image: url("./assets/sunny.jpg");
  background-size: cover;
  background-position: center;
  transition: background-image 0.4s ease;
}
#app.warm {
  background-image: url("./assets/sunny.jpg");
}
#app.cold {
  background-image: url("./assets/night.png");
}
main {
  min-height: 100vh;
  padding: 25px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 16px;
  box-shadow: 0px 4px 20px rgba(0, 0, 0, 0.1);
}
.search-box {
  width: 100%;
  max-width: 500px;
  margin-bottom: 30px;
}
.search-box .search-bar {
  width: 100%;
  padding: 15px;
  font-size: 18px;
  border: none;
  border-radius: 30px;
  outline: none;
  transition: box-shadow 0.4s ease;
  background-color: #fff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.search-box .search-bar:focus {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}
.location-box {
  text-align: center;
  margin-bottom: 20px;
}
.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 600;
  text-shadow: 2px 2px rgba(0, 0, 0, 0.2);
}
.location-box .date {
  color: #f1f1f1;
  font-size: 18px;
  font-weight: 300;
  margin-top: 5px;
}
.weather-box {
  text-align: center;
  background-color: rgba(255, 255, 255, 0.2);
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.2);
}
.weather-box .weather-icon {
  width: 100px; /* Adjust the size as needed */
  height: auto;
}
.data-row {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  margin-top: 20px;
}
.data-cube {
  background-color: rgba(255, 255, 255, 0.5);
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  flex: 1 1 150px; /* Adjust the size of the cubes */
  text-align: center;
}
.data-cube .data-title {
  font-size: 14px;
  font-weight: 600;
  color: #333;
}
.data-cube .data-value {
  font-size: 18px;
  font-weight: 700;
  color: #555;
}
</style>
