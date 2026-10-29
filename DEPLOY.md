# Deploying to PythonAnywhere (free)

PythonAnywhere suits this project: it runs Django directly, gives you a free `yourname.pythonanywhere.com` address, and keeps the SQLite file between restarts.
Their UI changes over time, so if a step looks different, follow their guide:
https://help.pythonanywhere.com/pages/DeployExistingDjangoProject

## How the project is configured

Production behaviour is controlled by environment variables, so no secrets live in the code.

| Variable | Meaning |
| --- | --- |
| `DJANGO_DEBUG` | `1` (default) for local work, `0` in production |
| `DJANGO_SECRET_KEY` | Required when `DJANGO_DEBUG=0`. The app refuses to start without it |
| `DJANGO_ALLOWED_HOSTS` | Comma-separated hosts, e.g. `yourname.pythonanywhere.com` |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | e.g. `https://yourname.pythonanywhere.com` |
| `DJANGO_DB_PATH` | Optional path for the SQLite file |

Generate a secret key with:

```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

## Steps

1. **Push the project to GitHub** (`.gitignore` already excludes the database and virtualenv).
2. **Create a free PythonAnywhere account** and open a **Bash console**.
3. **Clone and install:**
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   mkvirtualenv --python=/usr/bin/python3.12 student-venv   # use a Python version PythonAnywhere offers
   cd <repo-name>
   pip install -r requirements.txt
   ```
4. **Prepare the database and static files** (run inside the folder that contains `manage.py`):
   ```bash
   python manage.py migrate
   python manage.py seed_students      # optional demo data
   python manage.py createsuperuser    # the account you will log in with
   python manage.py collectstatic --noinput
   ```
5. **Create the web app:** Web tab, **Add a new web app**, choose **Manual configuration**, pick the same Python version as the virtualenv.
6. On the Web tab set:
   - **Virtualenv:** `student-venv`
   - **Static files:** URL `/static/`, directory `/home/<username>/<repo-name>/staticfiles`
7. **Edit the WSGI file** linked on the Web tab (not the `wsgi.py` inside the project). Replace its contents with:
   ```python
   import os
   import sys

   path = "/home/<username>/<repo-name>"
   if path not in sys.path:
       sys.path.append(path)

   os.environ["DJANGO_SETTINGS_MODULE"] = "config.settings"
   os.environ["DJANGO_DEBUG"] = "0"
   os.environ["DJANGO_SECRET_KEY"] = "<paste the generated key>"
   os.environ["DJANGO_ALLOWED_HOSTS"] = "<username>.pythonanywhere.com"
   os.environ["DJANGO_CSRF_TRUSTED_ORIGINS"] = "https://<username>.pythonanywhere.com"

   from django.core.wsgi import get_wsgi_application
   application = get_wsgi_application()
   ```
8. Click **Reload** on the Web tab and open `https://<username>.pythonanywhere.com`.

## After it is live

- Anyone can browse students and use the JSON API. Adding, editing and deleting require logging in with your superuser account.
- Do not paste your secret key into GitHub, the README or a screenshot.
- Free PythonAnywhere apps need to be renewed from the Web tab every so often, or they are disabled. Check their current rules.
- To update the site: `git pull` in the console, run `migrate` and `collectstatic` if needed, then press **Reload**.

## Using Render instead

The same code runs on Render (build command: `pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate`, start command: `gunicorn config.wsgi`) with the same environment variables. Be aware that SQLite files on Render's free tier are not kept between deploys, so your data and superuser would disappear. To use Render properly you would switch the database to PostgreSQL first.
