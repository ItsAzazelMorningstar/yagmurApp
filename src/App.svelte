<script>
  import axios from "axios";
  import {LocalNotifications} from "@capacitor/local-notifications"

  var currentHour = new Date().getHours();
  const apiKey = "7d117158d6204695a8a124719240612";
  const baseUrl = "https://api.weatherapi.com/v1/";
  const location = "Sultanhisar, Aydın";
  let rainMessage = "";

  let promisData = null;

  async function getWeather() {
    try {
      // API'den hava durumu verisini alıyoruz
      const response = await axios.get(`${baseUrl}forecast.json`, {
        params: {
          key: apiKey,
          q: location,
          days: 1,
          hourly: 24,
        },
      });

      // Gelen veriyi işleyelim
      const data = response.data;
      promisData = data;
      console.log(response);

      const currentRain = data.current.precip_mm > 0;
      const currentCondition = data.current.condition.text.toLowerCase();
      const dailyRain = data.forecast.forecastday[0].day.daily_will_it_rain;
      const dailyRainChance =
        data.forecast.forecastday[0].day.daily_chance_of_rain;
      console.log(currentHour);

      if (currentRain) {
        console.log("Şu anda yağmur yağıyor.");
        rainMessage = "Şu anda yağmur yağıyor.";
      } else if (
        currentCondition.includes("rain") ||
        currentCondition.includes("shower")
      ) {
        console.log("Şu anda yağmur var.");
        rainMessage = "Şu anda yağmur var.";
      } else if (dailyRain === 1) {
        console.log(
          `Bugün yağmur bekleniyor. Yağmur ihtimali: %${dailyRainChance}`
        );
        rainMessage = `Bugün yağmur bekleniyor. Yağmur ihtimali: %${dailyRainChance}`;
      } else {
        console.log("Bugün yağmur beklenmiyor.");
        rainMessage = "Bugün yağmur beklenmiyor.";
      }
      if (
        data.forecast.forecastday[0].hour[currentHour + 1] &&
        data.forecast.forecastday[0].hour[currentHour + 1].precip_mm > 0
      ) {
        console.log(`Saat ${currentHour + 1} da yağmur yağacak.`);
        bildirimGonder("Yağmur Geliyor","Sultan Hisara 1 Saat İçinde Yağmur Bekleniyor")
      }
      
      if (currentHour === 8) {
        if (dailyRain === 1) {
          bildirimGonder("Yağmur Geliyor",`Bugün yağmur bekleniyor. Yağmur ihtimali: %${dailyRainChance}`)
        }
      }
    } catch (error) {
      console.error("Hata:", error);
    }
  }
  

  // Hava durumu verisini al
  getWeather();
  setInterval(() => {
    getWeather();
  }, 3000000);




  function bildirimGonder(title,body) {
    let options= {notifications:[{title:title,body:body,id:1}]}
    LocalNotifications.schedule(options)
  }

</script>
{#if promisData !== null}
  <div
    style="width: 100%; display: flex; align-items: center; justify-content: center;"
  >
    <div
      style="display: flex; flex-direction: column; align-items: center; width: 100%; max-width: 1200px;padding-inline: 10px;"
    >
      <h1>Şu anki Saat: {currentHour}:00</h1>
      <br />
      <h2>Hizli Bilgi</h2>
      {rainMessage}
      <br />
      <h2 style="margin-bottom: 20px;">Saatlik Hava Durumu Bilgileri</h2>
      <div
        style="
        display: flex;
        justify-content: center;
        margin: 20px 0;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 8px;
        background-color: #f9f9f9;
        font-family: Arial, sans-serif;
        color: #333;
        box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
      "
      >
        <span style="margin-right: 20px;">
          <strong>Kullanım Bilgileri:</strong>
        </span>
        <div style="display: flex; align-items: center;">
          <div
            class="yuvarlak"
            style="
            background-color: lightblue;
            border: 1px solid #007bff;
            border-radius: 50%;
            margin-right: 5px;
          "
          ></div>
          <span class="kullanimtext">Yağmur yağacak saatler</span>
          <div
            class="yuvarlak"
            style="
            background-color: lightgreen;
            border: 1px solid #28a745;
            border-radius: 50%;
            margin-right: 5px;
          "
          ></div>
          <span>Şu anki saat</span>
        </div>
      </div>

      <div
        style="display: flex; flex-wrap: wrap; justify-content: center; gap: 20px;"
      >
        {#each promisData.forecast.forecastday[0].hour as item}
          {#if parseInt(item.time
              .split(" ")[1]
              .split(":")[0]) >= currentHour - 3}
            <div
              class="hour-card"
              style="box-shadow:inset black 0px 0px 0px {parseInt(
                item.time.split(' ')[1].split(':')[0]
              ) === currentHour
                ? '3px'
                : '0'};background-color: {item.precip_mm > 0
                ? '#A3D8F4'
                : '#E0E0E0'};"
            >
              <div class="time">
                <strong>Saat:</strong>
                {parseInt(item.time.split(" ")[1].split(":")[0]) < 10
                  ? `0${parseInt(item.time.split(" ")[1].split(":")[0])}`
                  : parseInt(item.time.split(" ")[1].split(":")[0])}:00
              </div>
              <div class="temperature">
                <strong>Sıcaklık:</strong>
                {item.temp_c}°C
              </div>
              <div class="feels-like">
                <strong>Hissedilen:</strong>
                {item.feelslike_c}°C
              </div>
              <div class="humidity"><strong>Nem:</strong> {item.humidity}%</div>
              <div class="rain-chance">
                <strong>Yağmur İhtimali:</strong>
                {item.precip_mm > 0 ? "Yağmur var" : "Yağmur yok"}
              </div>
            </div>
          {/if}
        {/each}
      </div>
    </div>
  </div>
{:else}
  Hava Durumu Bilgileri Alınıyor
{/if}

<style>
  * {
    user-select: none;
  }
  h1 {
    color: #333;
    font-size: 2rem;
    margin-bottom: 10px;
  }

  h2 {
    color: #555;
    font-size: 1.5rem;
    margin-bottom: 30px;
  }

  .hour-card {
    display: flex;
    flex-direction: column;
    margin: 10px;
    padding: 15px;
    border-radius: 10px;
    font-family: Arial, sans-serif;
    color: #333;
    width: 200px;
    text-align: center;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition:
      background-color 0.3s ease,
      transform 0.3s ease;
  }

  .hour-card:hover {
    transform: translateY(-5px);
  }

  .time,
  .temperature,
  .feels-like,
  .humidity,
  .rain-chance {
    margin: 5px 0;
  }

  .time {
    font-weight: bold;
    color: #333;
  }
  .yuvarlak {
    min-width: 16px;
    height: 16px;
  }
  .kullanimtext {
    margin-right: 15px;
  }
  .temperature,
  .feels-like,
  .humidity,
  .rain-chance {
    color: #555;
  }
  @media (max-width: 568px) {
    .hour-card {
      margin: 5px;
      padding: 10px;
      margin-inline: 23px;
      width: 100%;
    }
    .yuvarlak {
      height: 25px;
      min-width: 25px;
    }
    .kullanimtext {
      margin-right: 5px;
    }
  }
</style>
