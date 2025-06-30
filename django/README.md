# Django Backend Deployment on cPanel Shared Hosting (Subdomain Setup)

---

## 1. Prepare Your Django Backend for Deployment

### Update `settings.py`

* Load environment variables from `.env` using `dotenv`.
* Set:

  ```python
  DEBUG = False

  ALLOWED_HOSTS = ['api.asr25.com', 'localhost', '127.0.0.1']

  STATIC_URL = "/static/"
  STATIC_ROOT = os.path.join(BASE_DIR, "static/")

  MEDIA_URL = "/media/"
  MEDIA_ROOT = os.path.join(BASE_DIR, "media/")

  # Update other settings for production (databases, email, CORS, etc.)

  # Example of reading secrets from environment variables
  SECRET_KEY = os.getenv('DJANGO_SECRET_KEY', 'your_default_secret_key')

  # CORS: Allow all origins (adjust as needed)
  CORS_ALLOW_ALL_ORIGINS = True

  # Frontend URL for reference
  FRONTEND_SITE_URL = os.getenv("FRONTEND_SITE_URL", "https://asr25.com")
  ```
* Change any localhost URLs in your Django backend code or `.env` from:

  * `http://localhost:8000` → `https://api.asr25.com`
  * `http://localhost:3000` → `https://asr25.com`

---

## 2. Create the Subdomain in cPanel

* Go to **Domains > Subdomains**.
* Create the subdomain: `api.asr25.com`
* Set the document root to `public_html/api`
* This will serve your Django backend app.

---

## 3. Prepare and Upload Django Project Files

* Delete existing `__pycache__` folders and your local virtual environment.
* Compress your Django backend project folder into a `.zip` file.
* Using **cPanel File Manager**, upload and extract the `.zip` inside `public_html/api`.
* Your project root (where `manage.py` and `backend/` folder exist) should be inside `public_html/api`.

---

## 4. Create Python Application in cPanel

* Go to **Software > Setup Python App**.
* Click **Create Application**:

  * Python version: `3.8.20` (or your desired compatible version)
  * Application root: `public_html/api`
  * Application URL: `https://api.asr25.com`
* Click **Create**.

---

## 5. Create/Edit `passenger_wsgi.py` in `public_html/api`

Create or edit `passenger_wsgi.py` with the following content:

```python
import os
import sys
from pathlib import Path

# Debug log for passenger loading (optional)
try:
    with open("/home/asrcom/public_html/api/passenger_debug.log", "a") as f:
        f.write("Passenger loaded\n")
except Exception as e:
    print(f"Log write error: {e}", file=sys.stderr)

BASE_DIR = Path(__file__).resolve().parent
sys.path.insert(0, str(BASE_DIR))
sys.path.insert(0, str(BASE_DIR / "backend"))  # Adjust if your Django project folder is inside 'backend'

# Load environment variables from .env if it exists
env_path = BASE_DIR / ".env"
if env_path.exists():
    from dotenv import load_dotenv
    load_dotenv(dotenv_path=env_path)

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "backend.settings")

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

> **Note:**
> Replace `"backend.settings"` and the path in `sys.path.insert(0, str(BASE_DIR / "backend"))` according to your Django project folder name.

---

## 6. Install Dependencies & Run Management Commands

* Goto **Setup Python App** and click edit the app you created, use the provided `Execute python script` input field to run the commands one by one:

```bash
/home/asrcom/virtualenv/public_html/api/3.8/bin/pip install -r /home/asrcom/public_html/api/requirements.txt
```

```bash
/home/asrcom/public_html/api/manage.py migrate
```

```bash
/home/asrcom/public_html/api/manage.py collectstatic --noinput
```

---

## 7. Restart Python Application
* Click **Restart** to reload your Django backend.
