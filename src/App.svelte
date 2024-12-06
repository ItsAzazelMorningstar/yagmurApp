<script>
  import axios from "axios";

  var currentHour = new Date().getHours();
  const apiKey = "7d117158d6204695a8a124719240612";
  const baseUrl = "http://api.weatherapi.com/v1/";
  const location = "Sultanhisar, Aydın"; // Konumu ihtiyacınıza göre değiştirebilirsiniz.

  let promisData = null;

  async function getWeather() {
    try {
      // API'den hava durumu verisini alıyoruz
      const response = await axios.get(`${baseUrl}forecast.json`, {
        params: {
          key: apiKey,
          q: location,
          days: 1, // 1 günlük tahmin
          hourly: 24, // Saatlik verileri içerecek şekilde
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

      if (currentHour === 8) {
        if (currentRain) {
          console.log("Şu anda yağmur yağıyor.");
        } else if (
          currentCondition.includes("rain") ||
          currentCondition.includes("shower")
        ) {
          console.log("Şu anda yağmur var.");
        } else if (dailyRain === 1) {
          console.log(
            `Bugün yağmur bekleniyor. Yağmur ihtimali: %${dailyRainChance}`
          );
        } else {
          console.log("Bugün yağmur beklenmiyor.");
        }
      }
      if (
        data.forecast.forecastday[0].hour[currentHour + 1] &&
        data.forecast.forecastday[0].hour[currentHour + 1].precip_mm > 0
      ) {
        console.log(`Saat ${currentHour + 1} da yağmur yağacak.`);
      }
    } catch (error) {
      console.error("Hata:", error);
    }
  }

  // Hava durumu verisini al
  getWeather();
</script>

{#if promisData !== null}
  <div
    style="width: 100vw; display: flex; align-items: center; justify-content: center; padding: 20px;"
  >
    <div
      style="display: flex; flex-direction: column; align-items: center; width: 75%; max-width: 1200px;"
    >
      <h1>Şu anki Saat: {currentHour}:00</h1>
      <h2 style="margin-bottom: 20px;">Saatlik Yağmur İhtimalleri</h2>

      <div
        style="display: flex; flex-wrap: wrap; justify-content: center; gap: 20px;"
      >
        {#each promisData.forecast.forecastday[0].hour as item}
          <div
            style="display: flex; flex-direction: column; margin: 10px; padding: 15px; border-radius: 8px; font-family: Arial, sans-serif; color: #333; width: 200px; text-align: center; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);background-color: {parseInt(
              item.time.split(' ')[1].split(':')[0]
            ) === currentHour
              ? 'darkgray'
              : '#a9a9a9'};box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);"
          >
            <div><strong>Saat:</strong> {parseInt(item.time.split(' ')[1].split(':')[0]) < 10 
              ? `0${parseInt(item.time.split(' ')[1].split(':')[0])}` 
              : parseInt(item.time.split(' ')[1].split(':')[0])}:00</div>
            <div><strong>Sıcaklık:</strong> {item.temp_c}°C</div>
            <div><strong>Hissedilen:</strong> {item.feelslike_c}°C</div>
            <div><strong>Nem:</strong> {item.humidity}%</div>
            <div>
              <strong>Yağmur İhtimali:</strong>
              {item.precip_mm > 0 ? "Yağmur var" : "Yağmur yok"}
            </div>
          </div>
        {/each}
      </div>
    </div>
  </div>
{:else}
  Hava Durumu Bilgileri Alınıyor
{/if}

<style>
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
    background-color: #f4f4f4;
    border-radius: 8px;
    font-family: Arial, sans-serif;
    color: #333;
    width: 200px;
    text-align: center;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
</style>
