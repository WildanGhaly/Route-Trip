# HOS Trip Planner (Mono-repo)

Full-stack: Django + DRF backend and React + Vite frontend that plans a long-haul trip schedule under FMCSA HOS constraints, renders a map with markers, and draws day-by-day ELD-style logs.

Backend: /api/plan-trip/ (POST) — plans segments with pickup/drop, breaks, fuel, 14-hr window, 11-hr drive cap, 70-hr/8-day cycle (auto 34-hr reset), returns per-day segments + stops.

Frontend: form for inputs, live map, and ELD canvases. Dark/light auto, no modal popups.

## Repo layout
```
.
├─ backend/          # Django project (DRF API)
│  ├─ backend/       # settings, urls, wsgi/asgi
│  └─ tripplanner/   # serializers, views, hos logic, route helpers
└─ frontend/         # React + Vite app (TS)
   └─ src/           # components, api, canvas log, styles
```

## Quick start (local)

**Backend:**

```bash
cd Route-Trip-BE
python3 -m venv .venv && source .venv/bin/activate
pip install -U pip django djangorestframework django-cors-headers python-dotenv requests gunicorn
cp .env.example .env   # or create .env; see variables below
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

**Frontend:**

```bash
cd Route-Trip-FE
npm install
cp .env.example .env.local   # set VITE_API_BASE=http://127.0.0.1:8000
npm run dev                  # http://localhost:5173
```

## Production (one-liner view)

**Run backend with Gunicorn:**

```bash
cd backend
source .venv/bin/activate
gunicorn backend.wsgi:application --bind 127.0.0.1:8001 --workers 3
```

**Build frontend:**

```bash
cd frontend
# If serving FE and BE on the same domain via Nginx, set VITE_API_BASE= (empty)
npm run build
```

Nginx serves frontend/dist and proxies /api/ to 127.0.0.1:8001.

## API contract (summary)

**Endpoint:**

```
POST /api/plan-trip/
Content-Type: application/json
```

**Request body:**

```json
{
  "current_location": "Kansas City, MO",
  "pickup_location": "Denver, CO",
  "dropoff_location": "Los Angeles, CA",
  "current_cycle_used_hours": 42
}
```

**Response shape:**

```json
{
  "route": { "distance_mi": 1234.5, "duration_hr": 25.6, "polyline": null | "encoded_polyline" },
  "stops": [{ "type":"pickup|drop|fuel|break", "eta":"ISO", "duration_min":30 }],
  "days": [
    {
      "index": 1,
      "date": "2025-09-14",
      "segments": [
        { "t0":"08:00","t1":"09:00","status":"on_duty","label":"Pickup" },
        { "t0":"09:00","t1":"13:30","status":"driving","label":"" }
      ],
      "notes": "Day total: 10.0h driving; window used: 12.5h"
    }
  ]
}
```

## Environment

**Backend .env:**

```
SECRET_KEY=change-me
DEBUG=0
ALLOWED_HOSTS=your.domain,127.0.0.1,localhost
# If FE is on another origin:
CORS_ALLOW_ORIGINS=https://your-fe.example.com
ORS_API_KEY=your_openrouteservice_key
```

**Frontend .env.local:**

```
VITE_API_BASE=http://127.0.0.1:8000
```

## Notes

Geocoding (pickup/drop) uses public Nominatim; ORS (if key present) provides real driving distance/duration (polyline optional). Otherwise a simple fallback distance is computed and time ≈ distance / 50 mph.

Dark/light theme follows system; ELD canvas colors adapt via CSS variables.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.