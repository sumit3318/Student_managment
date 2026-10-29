# Student Management System (Django)

A small Django web app for managing students and their marks. It is a rebuild of my Python/MySQL Student Management System using Django's ORM, class-based views, templates and admin.

**Stack:** Python, Django, SQL (SQLite by default), JSON API

## Features

- Create, view, edit and delete students (roll no and email are unique)
- Record marks per subject; one entry per subject per student, marks cannot exceed the maximum
- Average percentage and grade (A+ to F) calculated per student
- Search by name, roll number or email, filter by course, paginated list
- Read-only JSON API for students and their marks
- Login protection: anyone can browse, but adding, editing and deleting need a logged-in user
- Django admin with inline marks editing
- Production-ready settings (environment variables, WhiteNoise, gunicorn), see [DEPLOY.md](DEPLOY.md)
- 21 automated tests (models, views, validation, login protection, JSON API)

## Run it locally

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

python manage.py migrate
python manage.py seed_students   # optional demo data
python manage.py createsuperuser # needed to log in and edit data
python manage.py runserver
```

Open http://127.0.0.1:8000/ and log in with the superuser to add or edit students.

## Run the tests

```bash
python manage.py test
```

## Pages and endpoints

| URL | What it does |
| --- | --- |
| `/` | Student list with search, course filter and pagination |
| `/accounts/login/` | Log in (required for the pages below that change data) |
| `/students/add/` | Add a student |
| `/students/<id>/` | Student detail, marks, add/remove marks |
| `/students/<id>/edit/` | Edit a student |
| `/students/<id>/delete/` | Delete a student (confirmation page) |
| `/api/students/` | JSON list. Supports `?q=` and `?course=` |
| `/api/students/<id>/` | JSON detail including marks |
| `/admin/` | Django admin |

Example JSON response from `/api/students/1/`:

```json
{
  "id": 1,
  "roll_no": "2022001",
  "name": "Aarav Sharma",
  "email": "aarav1@example.com",
  "course": "IT",
  "semester": 2,
  "average_percentage": 59.25,
  "grade": "D",
  "marks": [
    {"subject": "DBMS", "marks_obtained": 70.0, "max_marks": 100.0, "percentage": 70.0},
    {"subject": "Mathematics", "marks_obtained": 38.0, "max_marks": 100.0, "percentage": 38.0},
    {"subject": "Operating Systems", "marks_obtained": 66.0, "max_marks": 100.0, "percentage": 66.0},
    {"subject": "Python Programming", "marks_obtained": 63.0, "max_marks": 100.0, "percentage": 63.0}
  ]
}
```

## Project layout

```
config/                 project settings and root URLs
students/
  models.py             Student and Mark models, average/grade logic
  forms.py              ModelForms and the duplicate-subject validation
  views.py              class-based CRUD views, mark actions, JSON API
  urls.py               app routes
  admin.py              admin with inline marks
  templates/students/   HTML templates
  tests.py              test suite
  management/commands/  seed_students demo data command
```

## Things I would add next

- Per-user permissions (right now any logged-in user can edit everything)
- CSV export of results
- Switch the database to PostgreSQL or MySQL for larger deployments
