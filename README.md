Name: Yendluri Chandana
Reg: 212223100063


# weather-website(API)
html code:
```
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Simple Weather App</title>
  <style>
    :root{
      --bg: #0f1724;
      --card: #111827;
      --accent: #3b82f6;
      --muted: #9ca3af;
      --glass: rgba(255,255,255,0.04);
    }
    html,body{height:100%;}
    body{
      margin:0;display:flex;align-items:center;justify-content:center;
      font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;
      background: radial-gradient(1200px 400px at 10% 10%, rgba(59,130,246,0.08), transparent),
                  linear-gradient(180deg,#071021 0%, #071424 100%);
      color:#e6eef8;
      padding:24px;
    }
    .wrap{width:100%;max-width:760px}

    header{display:flex;align-items:center;gap:16px;margin-bottom:18px}
    .logo{width:52px;height:52px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;font-weight:700;color:white}
    h1{font-size:20px;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:13px}

    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
          border-radius:14px;padding:18px;box-shadow:0 6px 30px rgba(2,6,23,0.6);}

    form{display:flex;gap:10px;align-items:center}
    input[type=text]{flex:1;padding:12px 14px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:var(--glass);color:inherit;font-size:15px}
    button{padding:11px 16px;border-radius:10px;border:none;background:var(--accent);color:white;font-weight:600;cursor:pointer}
    button:disabled{opacity:.6;cursor:default}

    .result{margin-top:18px;display:grid;grid-template-columns:1fr 120px;gap:16px;align-items:center}
    .left{min-height:110px}
    .place{font-size:16px;font-weight:700}
    .time{color:var(--muted);font-size:13px;margin-top:4px}
    .temp{font-size:44px;font-weight:800;margin-top:8px}
    .desc{display:flex;gap:10px;align-items:center;margin-top:8px;color:var(--muted)}
    .condition-icon{width:84px;height:84px;display:flex;align-items:center;justify-content:center;background:rgba(255,255,255,0.02);border-radius:14px}
    .meta{display:flex;gap:14px;margin-top:12px;color:var(--muted);font-size:13px}

    .error{margin-top:14px;color:#ffb4b4;background:rgba(255,0,0,0.03);padding:10px;border-radius:8px}

    footer{margin-top:12px;color:var(--muted);font-size:13px}

    @media (max-width:520px){.result{grid-template-columns:1fr;}.condition-icon{width:64px;height:64px}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="logo">W</div>
      <div>
        <h1>Simple Weather App</h1>
        <p class="lead">Type a city, town or ZIP/postal code and see the current temperature.</p>
      </div>
    </header>

    <div class="card">
      <form id="searchForm" autocomplete="off">
        <input id="locationInput" type="text" placeholder="Enter location (e.g. London or 10001)" required />
        <button id="searchBtn" type="submit">Get Weather</button>
      </form>

      <div id="output"></div>
      <div id="error" class="error" style="display:none"></div>

      <footer>Note: This demo calls WeatherAPI directly from the browser using the API key you supplied. Exposing API keys in client-side code is insecure. For production, proxy requests through your server.</footer>
    </div>
  </div>

  <script>
    // === CONFIG ===
    // Using the API key you provided. If you prefer to replace it, edit below.
    const WEATHER_API_KEY = '1c1db83fb3334c909b4152120252510';
    const BASE_URL = 'http://api.weatherapi.com/v1/current.json';

    const form = document.getElementById('searchForm');
    const input = document.getElementById('locationInput');
    const output = document.getElementById('output');
    const errEl = document.getElementById('error');
    const btn = document.getElementById('searchBtn');

    function showError(msg){
      errEl.style.display = 'block';
      errEl.textContent = msg;
    }
    function clearError(){ errEl.style.display = 'none'; errEl.textContent = ''; }

    function renderWeather(data){
      const loc = data.location;
      const cur = data.current;

      const html = `
        <div class="result">
          <div class="left">
            <div class="place">${escapeHtml(loc.name)}, ${escapeHtml(loc.region || loc.country)}</div>
            <div class="time">Local time: ${escapeHtml(loc.localtime)}</div>
            <div class="temp">${Math.round(cur.temp_c)}°C / ${Math.round(cur.temp_f)}°F</div>
            <div class="desc">
              <div style="display:flex;flex-direction:column">
                <div style="font-weight:600">${escapeHtml(cur.condition.text)}</div>
                <div class="meta">Humidity: ${cur.humidity}% &nbsp; • &nbsp; Wind: ${cur.wind_kph} kph</div>
              </div>
            </div>
            <div class="meta">Air Quality Index (US): ${cur.air_quality && cur.air_quality['us-epa-index'] ? cur.air_quality['us-epa-index'] : 'N/A'}</div>
          </div>
          <div class="condition-icon">
            <img src="${escapeHtml(cur.condition.icon)}" alt="icon" width="64" height="64" />
          </div>
        </div>
      `;
      output.innerHTML = html;
    }

    function escapeHtml(s){
      return String(s)
        .replace(/&/g,'&amp;')
        .replace(/</g,'&lt;')
        .replace(/>/g,'&gt;')
        .replace(/"/g,'&quot;')
        .replace(/'/g,'&#039;');
    }

    async function fetchWeather(q){
      clearError();
      output.innerHTML = '';
      btn.disabled = true; btn.textContent = 'Loading...';

      // Build URL
      const url = `${BASE_URL}?key=${encodeURIComponent(WEATHER_API_KEY)}&q=${encodeURIComponent(q)}&aqi=yes`;

      try{
        const res = await fetch(url);
        if(!res.ok){
          const txt = await res.text();
          throw new Error(`Server returned ${res.status} — ${txt}`);
        }
        const data = await res.json();
        if(data && data.error){
          throw new Error(data.error.message || 'Unknown API error');
        }
        renderWeather(data);
      }catch(err){
        console.error(err);
        showError('Could not fetch weather: ' + err.message);
      }finally{
        btn.disabled = false; btn.textContent = 'Get Weather';
      }
    }

    form.addEventListener('submit', e =>{
      e.preventDefault();
      const q = input.value.trim();
      if(!q) { showError('Please enter a location'); return; }
      fetchWeather(q);
    });

    // Quick convenience: allow pressing Enter in input to submit
    input.addEventListener('keydown', e => {
      if(e.key === 'Enter'){
        e.preventDefault();
        form.requestSubmit();
      }
    });

    // Optional: prefill with a friendly example
    input.value = 'London';
    // Auto-fetch initial weather for prefilled value (comment out if undesired)
    // fetchWeather(input.value);
  </script>
</body>
</html>
```
output:
<img width="1259" height="851" alt="image" src="https://github.com/user-attachments/assets/e7b4a09a-bcba-4b04-add7-a5a96fa87dc6" />

<img width="1257" height="861" alt="image" src="https://github.com/user-attachments/assets/a7b60238-fcca-49d4-ab51-6fce47455d8b" />
