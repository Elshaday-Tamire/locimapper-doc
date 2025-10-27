# Locimapper Developer Quickstart

## 1) Navigate to Developer Settings
**Side menu → Developer Settings.**  
Create a new **App**.

- **App Credentials:** copy the **Client ID** and **Client Secret** (the secret is shown **once**). Store server-side only.
- **Branding:** set UI colors/logo so the hosted Import & Validator screens match your app.

---

## 2) Create Integrations
From your App:
- Create an **Integration** (reusable), or a **Schema-specific Integration** (tied to a schema).
- Save the resulting **`integration_id`**.

---

## 3) Launch the Import/Validation Page (Server-to-Server)
Your backend calls the launcher endpoint, authenticates with your app credentials, and receives a **single-use, short-lived** URL for the hosted page.

**Endpoint**
```
POST /accounts/integrations/launch
```

**Request (JSON)**
```json
{
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET",
  "integration_id": "YOUR_INTEGRATION_ID"
}
```

**Successful Response (200)**
```json
{
  "frontend_url": "https://example.com/import-standalone#<short-lived-token>"
}
```

**Notes**
- The token is in the **URL fragment** (after `#`) and is **single-use** and **short-lived**.
- Typical error: **422** (validation error) with details.

---

## 4) Redirect or Open in a Modal (iframe)
You can redirect users to `frontend_url`, **or** open it in a modal iframe.

**Recommended iframe size:** `{ w: 900, h: 700 }`.

**Minimal Modal + iframe (vanilla JS/HTML)**
```html
<!doctype html>
<html>
<body>
  <button id="open">Open Import</button>

  <div id="backdrop" style="display:none;position:fixed;inset:0;background:rgba(0,0,0,.4);align-items:center;justify-content:center;">
    <div style="background:#fff;position:relative;padding:8px;border-radius:8px;">
      <button onclick="closeModal()" style="position:absolute;top:8px;right:8px;">✕</button>
      <iframe id="lm-frame" width="900" height="700" style="border:0"></iframe>
    </div>
  </div>

  <script>
    async function getFrontendUrl() {
      // Call YOUR backend that wraps the POST /accounts/integrations/launch
      const res = await fetch('/api/locimapper/launch', { method: 'POST' });
      const { frontend_url } = await res.json();
      return frontend_url;
    }
    function openModal(url) {
      document.getElementById('lm-frame').src = url;
      document.getElementById('backdrop').style.display = 'flex';
    }
    function closeModal() {
      document.getElementById('lm-frame').src = 'about:blank';
      document.getElementById('backdrop').style.display = 'none';
    }
    document.getElementById('open').onclick = async () => openModal(await getFrontendUrl());
  </script>
</body>
</html>
```

---

## Example Backend Calls

**cURL**
```bash
curl -X POST https://YOUR_API/accounts/integrations/launch   -H "Content-Type: application/json"   -d '{
    "client_id":"YOUR_CLIENT_ID",
    "client_secret":"YOUR_CLIENT_SECRET",
    "integration_id":"YOUR_INTEGRATION_ID"
  }'
```

**Node (Express)**
```js
app.post('/api/locimapper/launch', async (req, res) => {
  const payload = {
    client_id: process.env.LOCIMAPPER_CLIENT_ID,
    client_secret: process.env.LOCIMAPPER_CLIENT_SECRET,
    integration_id: process.env.LOCIMAPPER_INTEGRATION_ID
  };
  const r = await fetch(process.env.LOCIMAPPER_BASE + '/accounts/integrations/launch', {
    method: 'POST',
    headers: {'Content-Type':'application/json'},
    body: JSON.stringify(payload)
  });
  res.status(r.status).json(await r.json());
});
```

**Python (FastAPI)**
```python
@app.post("/api/locimapper/launch")
async def lm_launch():
  payload = {
    "client_id": os.getenv("LOCIMAPPER_CLIENT_ID"),
    "client_secret": os.getenv("LOCIMAPPER_CLIENT_SECRET"),
    "integration_id": os.getenv("LOCIMAPPER_INTEGRATION_ID"),
  }
  async with httpx.AsyncClient() as client:
    r = await client.post(
      f"{os.getenv('LOCIMAPPER_BASE')}/accounts/integrations/launch",
      json=payload, timeout=20.0
    )
  return JSONResponse(status_code=r.status_code, content=r.json())
```

---

## Security Checklist
- Never expose the **client secret** in frontend code or logs.
- Keep credentials in a secure secret store; rotate if leaked.
- Only call the launcher from your **backend**.
- Treat the returned URL as **single-use**; fetch a new one per session/flow.

---

✅ That’s it — your users can now launch **Locimapper’s Mapping & Validation UI** from inside your app, styled with your brand, in a clean `900×700` modal.
