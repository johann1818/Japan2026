# Japan2026
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>2026 日本美食縱走</title>
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">

<style>
  body { font-family: 'Inter', sans-serif; -webkit-overflow-scrolling: touch; background-color: #F3F4F6; overscroll-behavior-y: none; }
  .no-scrollbar::-webkit-scrollbar { display: none; }
  .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
  .calc-btn:active { transform: scale(0.95); background-color: #e5e7eb; }
  
  /* 載入動畫 */
  @keyframes spin { to { transform: rotate(360deg); } }
  .animate-spin-slow { animation: spin 3s linear infinite; }
</style>
</head>

<body class="text-gray-800 select-none">

<div id="app" class="max-w-md mx-auto bg-[#F5F7FA] min-h-screen relative shadow-2xl overflow-hidden pb-24">

  <header class="bg-white pt-12 pb-4 px-4 sticky top-0 z-40 shadow-sm">
    <div class="flex justify-between items-center mb-4">
      <div>
          <h1 class="text-xl font-extrabold tracking-tight flex items-center text-slate-800">
              <span class="mr-2 text-2xl">🇯🇵</span> 2026 日本縱走
          </h1>
          <p class="text-xs text-blue-600 font-bold mt-1 ml-1">
             <i class="fas fa-clock mr-1"></i> 距離出發還有 {{ daysLeft }} 天
          </p>
      </div>
      <div class="text-xs font-bold text-gray-500 bg-gray-100 px-2 py-1 rounded-full flex items-center gap-1">
         <div class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></div> LIVE
      </div>
    </div>

    <div class="flex overflow-x-auto gap-3 pb-2 no-scrollbar scroll-smooth">
      <div v-for="(day, index) in days" :key="index" 
           @click="switchDay(index)"
           class="flex-shrink-0 px-4 py-3 rounded-2xl cursor-pointer transition-all border flex flex-col items-center justify-center min-w-[4.5rem]"
           :class="currentDayIndex === index 
             ? 'bg-blue-600 text-white border-blue-600 shadow-lg shadow-blue-200' 
             : 'bg-white text-gray-400 border-gray-100'">
        <span class="text-[10px] uppercase font-bold tracking-wider opacity-80 pointer-events-none">{{ day.dateShort }}</span>
        <span class="text-lg font-bold leading-none mt-1 pointer-events-none">{{ day.dayNum }}</span>
      </div>
    </div>
  </header>

  <main class="p-4 relative z-0">
    
    <div class="bg-gradient-to-r from-blue-500 to-indigo-600 rounded-3xl p-5 text-white shadow-lg mb-6 flex justify-between items-center relative overflow-hidden">
        <div class="absolute right-[-20px] top-[-20px] w-32 h-32 bg-white opacity-10 rounded-full blur-xl"></div>
        <div class="z-10">
            <h2 class="text-lg font-bold flex items-center gap-2">
                <i class="fas fa-map-marker-alt text-blue-200"></i> {{ days[currentDayIndex].city }}
            </h2>
            <p class="text-blue-100 text-xs mt-1 opacity-90">當地即時氣溫</p>
        </div>
        <div class="text-center z-10 flex flex-col items-end">
            <div v-if="loadingWeather" class="text-sm animate-pulse">Updating...</div>
            <div v-else class="flex items-center gap-3">
                <i class="fas text-3xl" :class="weatherData.icon"></i>
                <span class="text-4xl font-bold tracking-tighter">{{ weatherData.temp }}°</span>
            </div>
            <p v-if="!loadingWeather" class="text-xs text-blue-100 mt-1">{{ weatherData.desc }}</p>
        </div>
    </div>

    <div class="space-y-4">
      <div v-for="(event, idx) in days[currentDayIndex].events" :key="idx" 
           class="bg-white p-4 rounded-2xl shadow-sm border border-gray-50 flex gap-4 relative group hover:shadow-md transition-all active:scale-[0.98] duration-100"
           @click="event.location ? openMapModal(event) : null">
        
        <div class="flex-shrink-0 pt-1">
            <div class="w-12 h-12 rounded-2xl flex items-center justify-center text-xl shadow-inner"
                 :class="{
                    'bg-blue-50 text-blue-500': event.type === 'flight',
                    'bg-gray-100 text-gray-500': event.type === 'hotel' || event.type === 'transport',
                    'bg-pink-50 text-pink-500': event.type === 'food',
                    'bg-emerald-50 text-emerald-500': event.type === 'sightseeing'
                 }">
                 <span v-if="event.type === 'flight'">✈️</span>
                 <span v-else-if="event.type === 'hotel'">🛏️</span>
                 <span v-else-if="event.type === 'transport'">🚌</span>
                 <span v-else-if="event.type === 'food'">🍜</span>
                 <span v-else>🏔️</span>
            </div>
        </div>

        <div class="flex-grow pr-8">
            <div class="flex items-center gap-2 mb-1">
                <span class="font-bold text-blue-600 font-mono text-sm bg-blue-50 px-2 py-0.5 rounded">{{ event.time }}</span>
                <span class="text-[10px] text-gray-400 border border-gray-200 px-1.5 rounded" v-if="event.tag">{{ event.tag }}</span>
            </div>
            <h3 class="font-bold text-gray-800 text-[15px] leading-snug">{{ event.title }}</h3>
            <p v-if="event.desc" class="text-gray-500 text-xs mt-1">{{ event.desc }}</p>
            <div v-if="event.location" class="mt-2 flex items-center gap-1 text-[11px] text-gray-400">
                <i class="fas fa-map-pin"></i> {{ event.location }}
            </div>
        </div>

        <button v-if="event.location" 
                class="absolute right-4 top-1/2 -translate-y-1/2 w-10 h-10 bg-gray-100 rounded-full flex items-center justify-center text-gray-500">
            <i class="fas fa-map-marked-alt text-lg"></i>
        </button>
      </div>
    </div>
  </main>

  <div class="fixed bottom-6 right-6 z-50 flex flex-col gap-3">
    <button @click.stop="showCalc = !showCalc" class="w-14 h-14 bg-emerald-500 text-white rounded-full shadow-xl shadow-emerald-200 flex items-center justify-center text-xl hover:scale-105 active:scale-95 transition-transform group">
      <i class="fas fa-calculator" :class="{'animate-spin-slow': loadingRate}"></i>
    </button>
  </div>

  <transition name="fade">
    <div v-if="showCalc" class="fixed inset-0 bg-black/40 z-[60] flex items-end justify-center sm:items-center p-4 backdrop-blur-[2px]" @click.self="showCalc = false">
      <div class="bg-white w-full max-w-sm rounded-3xl p-5 shadow-2xl">
        
        <div class="flex justify-between items-center mb-4">
            <div>
                <h3 class="font-bold text-lg text-gray-700">即時匯率試算</h3>
                <p class="text-[10px] text-gray-400">
                    <span v-if="loadingRate"><i class="fas fa-sync fa-spin"></i> 更新匯率中...</span>
                    <span v-else>1 JPY = {{ exchangeRate }} TWD</span>
                </p>
            </div>
            <button @click="showCalc = false" class="text-gray-400 hover:text-gray-600"><i class="fas fa-times"></i></button>
        </div>

        <div class="bg-gray-100 p-4 rounded-2xl mb-4 text-right">
            <div class="text-gray-500 text-xs font-bold mb-1">¥ 日幣金額</div>
            <input type="number" v-model="yenAmount" class="w-full bg-transparent text-3xl font-bold text-gray-800 text-right outline-none placeholder-gray-300" placeholder="0">
        </div>

        <div class="bg-blue-50 p-4 rounded-2xl mb-6 text-right border border-blue-100">
            <div class="text-blue-500 text-xs font-bold mb-1">NT$ 台幣</div>
            <div class="text-3xl font-bold text-blue-600">{{ convertedTwd }}</div>
        </div>
        
        <div class="grid grid-cols-3 gap-2">
           <button v-for="n in [7,8,9,4,5,6,1,2,3]" :key="n" @click="appendNum(n)" class="calc-btn py-3 bg-gray-50 rounded-xl font-bold text-lg text-gray-600 shadow-sm border border-gray-100">{{ n }}</button>
           <button @click="yenAmount = ''" class="calc-btn py-3 bg-red-50 text-red-500 rounded-xl font-bold shadow-sm border border-red-100">C</button>
           <button @click="appendNum(0)" class="calc-btn py-3 bg-gray-50 rounded-xl font-bold text-lg text-gray-600 shadow-sm border border-gray-100">0</button>
           <button @click="showCalc = false" class="calc-btn py-3 bg-blue-600 text-white rounded-xl font-bold shadow-sm shadow-blue-200"><i class="fas fa-check"></i></button>
        </div>

      </div>
    </div>
  </transition>

  <transition name="fade">
    <div v-if="showModal" class="fixed inset-0 bg-black/60 z-[70] flex items-center justify-center p-4 backdrop-blur-sm" @click="closeModal">
        <div class="bg-white w-full max-w-sm rounded-3xl overflow-hidden shadow-2xl relative h-[400px] flex flex-col" @click.stop>
            <div class="p-3 bg-gray-50 border-b flex justify-between items-center">
                <h3 class="font-bold text-sm truncate pr-2">{{ selectedEvent?.location }}</h3>
                <button @click="closeModal"><i class="fas fa-times text-gray-400"></i></button>
            </div>
            <iframe width="100%" height="100%" class="flex-grow" :src="mapUrl" frameborder="0" allowfullscreen></iframe>
            <button @click="navigateTo(selectedEvent?.location)" class="m-3 py-3 rounded-xl bg-blue-600 text-white font-bold shadow-lg flex items-center justify-center gap-2">
                <i class="fas fa-location-arrow"></i> 開啟 Google Maps
            </button>
        </div>
    </div>
  </transition>

</div>

<script>
  const { createApp, ref, computed, onMounted } = Vue;

  createApp({
    setup() {
      const currentDayIndex = ref(0);
      const loadingWeather = ref(false);
      const weatherData = ref({ temp: '--', icon: 'fa-minus', desc: '載入中' });
      const showModal = ref(false);
      const selectedEvent = ref(null);
      const mapUrl = ref('');
      const showCalc = ref(false);
      const yenAmount = ref('');
      const exchangeRate = ref(0.22); 
      const loadingRate = ref(false);
      
      const convertedTwd = computed(() => {
          if(!yenAmount.value) return '0';
          return Math.round(yenAmount.value * exchangeRate.value).toLocaleString();
      });

      const fetchExchangeRate = async () => {
        loadingRate.value = true;
        try {
            const res = await fetch('https://api.frankfurter.app/latest?from=JPY&to=TWD');
            const data = await res.json();
            if(data && data.rates && data.rates.TWD) {
                exchangeRate.value = data.rates.TWD;
            }
        } catch (e) {
            console.error("匯率更新失敗", e);
        } finally {
            loadingRate.value = false;
        }
      };

      const days = ref([
        {
          dateShort: '1/17', dayNum: 'D1', city: 'Tokyo', lat: 35.6895, lon: 139.6917,
          events: [
            { time: '16:50', title: 'NH854 飛往羽田', type: 'flight', desc: 'ANA (座位 23A)', location: '台北松山機場', tag: '飛行' },
            { time: '21:30', title: '入住大田區 Airbnb', type: 'hotel', desc: '房東: サニータウン', location: '東京都大田區仲六鄉1-23-5' }
          ]
        },
        {
          dateShort: '1/18', dayNum: 'D2', city: 'Tokyo', lat: 35.6895, lon: 139.6917,
          events: [
            { time: '12:00', title: 'Shake Shack 新宿店', type: 'food', desc: '午餐', location: 'Shake Shack Shinjuku', tag: '午餐' },
            { time: '19:00', title: '肉屋の台所', type: 'food', desc: '和牛燒肉吃到飽', location: '肉屋の台所 渋谷宮益坂店', tag: '晚餐' },
            { time: '21:00', title: 'SHIBUYA SKY', type: 'sightseeing', desc: '露天夜景', location: 'SHIBUYA SKY' }
          ]
        },
        {
          dateShort: '1/19', dayNum: 'D3', city: 'Sapporo', lat: 43.0618, lon: 141.3545,
          events: [
            { time: '15:00', title: 'NH69 飛往札幌', type: 'flight', desc: 'ANA (座位 32C)', location: '羽田機場 第二航廈' },
            { time: '18:00', title: '入住 京王 Prelia', type: 'hotel', desc: '預約代號: TF05F6', location: 'KEIO PRELIA HOTEL SAPPORO' },
            { time: '19:30', title: 'にく久 (肉割烹)', type: 'food', desc: '預約代號: SD033', location: '大衆肉割烹 にく久 札幌店' }
          ]
        },
        {
          dateShort: '1/20', dayNum: 'D4', city: 'Otaru', lat: 43.1907, lon: 140.9947,
          events: [
            { time: '10:00', title: '前往小樽', type: 'transport', desc: 'JR 快速 Airport', location: '札幌駅' },
            { time: '16:00', title: 'LeTAO 下午茶', type: 'food', desc: '雙層起司蛋糕', location: '小樽洋菓子舗 LeTAO 本店' }
          ]
        },
        {
          dateShort: '1/21', dayNum: 'D5', city: 'Biei', lat: 43.5905, lon: 142.4686,
          events: [
            { time: '09:20', title: 'KKday 行程集合', type: 'transport', desc: '大通站出口31', location: 'Sapporo Odori Station Exit 31', tag: '集合' },
            { time: '16:30', title: '白鬚瀑布 & 青池', type: 'sightseeing', desc: '冬季點燈', location: '白ひげの滝' }
          ]
        },
        {
          dateShort: '1/22', dayNum: 'D6', city: 'Sapporo', lat: 43.0618, lon: 141.3545,
          events: [
            { time: '13:30', title: 'Es Con Field', type: 'sightseeing', desc: '火腿隊主場', location: 'Es Con Field Hokkaido' },
            { time: '18:00', title: '禅 (壽喜燒)', type: 'food', desc: '預約代號: SD033', location: '牛しゃぶ・すき焼専門店 禅 札幌' }
          ]
        },
        {
          dateShort: '1/23', dayNum: 'D7', city: 'Sapporo', lat: 43.0618, lon: 141.3545,
          events: [
            { time: '10:00', title: '札幌自由活動', type: 'sightseeing', desc: '購物', location: '狸小路商店街' },
            { time: '18:00', title: 'ふくすけ (羊肉)', type: 'food', desc: '預約代號: 8GA9', location: '北海道産羊・野菜ふくすけ' }
          ]
        },
        {
          dateShort: '1/24', dayNum: 'D8', city: 'Jozankei', lat: 42.9649, lon: 141.1611,
          events: [
            { time: '09:30', title: '前往定山溪', type: 'transport', desc: '河童號巴士', location: '札幌駅バスターミナル' },
            { time: '11:00', title: '豐平峽溫泉', type: 'food', desc: '印度烤餅', location: '豐平峽溫泉' }
          ]
        },
        {
          dateShort: '1/25', dayNum: 'D9', city: 'Sapporo', lat: 43.0618, lon: 141.3545,
          events: [
            { time: '14:00', title: '入住 Vessel Hotel', type: 'hotel', desc: '代號: 3118-06', location: 'Vessel Hotel Campana Susukino' }
          ]
        },
        {
          dateShort: '1/26', dayNum: 'D10', city: 'Tokyo', lat: 35.6895, lon: 139.6917,
          events: [
            { time: '15:30', title: 'CTS 飛往 HND', type: 'flight', desc: 'ANA (座位 33A)', location: '新千歲機場' },
            { time: '19:00', title: '入住蒲田 Airbnb', type: 'hotel', desc: '房東: NESTo', location: '東京都大田區蒲田4-28-5' }
          ]
        },
        {
          dateShort: '1/29', dayNum: 'D13', city: 'Tokyo', lat: 35.6895, lon: 139.6917,
          events: [
            { time: '09:20', title: 'HND 飛往 TPE', type: 'flight', desc: 'ANA (座位 23K)', location: '羽田機場 第二航廈' }
          ]
        }
      ]);

      const appendNum = (num) => { yenAmount.value = yenAmount.value.toString() + num.toString(); };
      
      const openMapModal = (event) => {
        selectedEvent.value = event;
        // 使用正確的 Google Maps Embed 網址
        mapUrl.value = `https://maps.google.com/maps?q=${encodeURIComponent(event.location)}&t=&z=15&ie=UTF8&iwloc=&output=embed`;
        showModal.value = true;
      };

      const closeModal = () => { showModal.value = false; mapUrl.value = ''; };
      
      const navigateTo = (location) => {
        // 使用 Google Maps Universal Link，這會自動喚醒手機上的 App
        window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(location)}`, '_blank');
      };

      const fetchWeather = async (lat, lon) => {
        loadingWeather.value = true;
        try {
            const res = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`);
            const data = await res.json();
            const code = data.current_weather.weathercode;
            const temp = Math.round(data.current_weather.temperature);
            let icon = 'fa-sun', desc = '晴朗';
            if(code >= 1 && code <= 3) { icon = 'fa-cloud-sun'; desc = '多雲'; }
            else if(code >= 71) { icon = 'fa-snowflake'; desc = '下雪'; } 
            else if(code >= 51) { icon = 'fa-cloud-rain'; desc = '有雨'; }
            weatherData.value = { temp, icon, desc };
        } catch { weatherData.value = { temp: '--', icon: 'fa-exclamation', desc: 'N/A' }; }
        finally { loadingWeather.value = false; }
      };

      const switchDay = (index) => {
        currentDayIndex.value = index;
        fetchWeather(days.value[index].lat, days.value[index].lon);
      };

      const today = new Date();
      const tripStart = new Date('2026-01-17');
      const daysLeft = Math.ceil((tripStart - today) / (1000 * 60 * 60 * 24));

      onMounted(() => {
        switchDay(0);
        fetchExchangeRate();
      });

      return {
        currentDayIndex, days, weatherData, loadingWeather, switchDay,
        showModal, selectedEvent, mapUrl, openMapModal, closeModal, navigateTo,
        showCalc, yenAmount, convertedTwd, exchangeRate, loadingRate, appendNum, daysLeft
      };
    }
  }).mount('#app');
</script>

</body>
</html>
