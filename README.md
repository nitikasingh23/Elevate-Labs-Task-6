# Elevate-Labs-Task-6

# 🌐 Portfolio Website — Built with Flask

A personal portfolio website built with **Python + Flask** featuring a dark editorial design, multi-page routing, Jinja2 HTML templates, and a fully working contact form with server-side validation and flash messaging.

---

## 🖥️ Live Preview Pages

| Page | Route | Description |
|---|---|---|
| Home | `/` | Hero, About strip, Skills, Featured Projects, CTA |
| About | `/about` | Full bio, Education timeline, Tech stack |
| Projects | `/projects` | All 6 projects with tech tags |
| Contact | `/contact` | Contact form with validation + flash messages |

---

## 📌 Features

| Feature | Implementation |
|---|---|
| 🧭 Multi-page routing | `@app.route()` decorators in Flask |
| 🎨 Jinja2 Templates | `base.html` extended by all pages |
| 📬 Contact Form | POST handler with server-side validation |
| 💾 Message Storage | Form submissions saved to `messages.json` |
| ⚡ Flash Messages | `flash()` + `get_flashed_messages()` for success/error |
| 📱 Responsive Design | CSS Grid + media queries, mobile nav toggle |
| 🎞️ Animations | CSS `@keyframes` + IntersectionObserver scroll reveals |
| 🔒 Input Validation | Name, email format, empty field checks |

---

## 📂 Project Structure

```
portfolio/
│
├── app.py                  # Flask app — routes, data, form logic
├── requirements.txt        # Python dependencies
├── messages.json           # Auto-created: stores contact form submissions
│
├── templates/
│   ├── base.html           # Master layout (navbar, footer, flash messages)
│   ├── index.html          # Home page
│   ├── about.html          # About & education page
│   ├── projects.html       # All projects page
│   └── contact.html        # Contact form page
│
└── static/
    ├── css/
    │   └── style.css       # Full design system (900+ lines)
    └── js/
        └── main.js         # Mobile nav + scroll animation observer
```

---

## 🚀 How to Run Locally

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/portfolio-flask.git
cd portfolio-flask
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Run the Flask App
```bash
python app.py
```

### 4. Open in Browser
```
http://127.0.0.1:5000
```

> ✅ No database needed. No external APIs. Just Flask + Python + HTML/CSS.

---

## 🖥️ Sample Terminal Output

```
 * Serving Flask app 'app'
 * Debug mode: on
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

---

## 💡 What I Did — Step-by-Step Explanation

---

### 🔹 1. Set Up Flask Routing

Flask uses the `@app.route()` decorator to map URLs to Python functions (called **view functions**). Each page is one route:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html", data=PORTFOLIO_DATA)

@app.route("/projects")
def projects():
    return render_template("projects.html", data=PORTFOLIO_DATA)

@app.route("/about")
def about():
    return render_template("about.html", data=PORTFOLIO_DATA)
```

The `data=PORTFOLIO_DATA` argument passes a Python dictionary to the HTML template so Jinja2 can render it dynamically.

---

### 🔹 2. Jinja2 Template Inheritance (`base.html`)

Instead of repeating the navbar and footer in every file, one `base.html` parent template holds the common structure. All other pages **extend** it:

**base.html** defines the skeleton:
```html
<!DOCTYPE html>
<html>
<head><title>{% block title %}{% endblock %}</title></head>
<body>
  <nav>...</nav>
  {% block content %}{% endblock %}  <!-- child fills this -->
  <footer>...</footer>
</body>
</html>
```

**index.html** inherits and fills in the block:
```html
{% extends "base.html" %}
{% block title %}Home — Sundar{% endblock %}

{% block content %}
  <section class="hero">...</section>
{% endblock %}
```

This is **DRY (Don't Repeat Yourself)** — change the navbar once and it updates everywhere.

---

### 🔹 3. Dynamic Data with Jinja2

All personal info (name, projects, skills, education) is stored as a Python dictionary in `app.py` and passed to templates. Jinja2 renders it using `{{ }}` and `{% %}` syntax:

```python
# app.py
PORTFOLIO_DATA = {
    "name": "Sundar",
    "projects": [
        {"title": "Calculator CLI", "tech": ["Python"], "icon": "🧮"},
        ...
    ]
}
```

```html
<!-- index.html -->
<h1>Hi, I'm {{ data.name }}</h1>

{% for project in data.projects %}
  <div class="project-card">
    <span>{{ project.icon }}</span>
    <h3>{{ project.title }}</h3>
  </div>
{% endfor %}
```

---

### 🔹 4. Contact Form with POST Handling

The contact page uses `methods=["GET", "POST"]` to handle both showing the form and receiving it:

```python
@app.route("/contact", methods=["GET", "POST"])
def contact():
    if request.method == "POST":
        name    = request.form.get("name", "").strip()
        email   = request.form.get("email", "").strip()
        message = request.form.get("message", "").strip()

        # Validate
        errors = []
        if not name:          errors.append("Name is required.")
        if "@" not in email:  errors.append("Valid email required.")
        if not message:       errors.append("Message cannot be empty.")

        if errors:
            for err in errors: flash(err, "error")
            return render_template("contact.html", ...)

        # Save to JSON file
        save_message(name, email, message)
        flash(f"Thanks {name}! Message received.", "success")
        return redirect(url_for("contact"))

    return render_template("contact.html", data=PORTFOLIO_DATA)
```

Key Flask concepts used:
- `request.method` — checks if it's GET (show form) or POST (submit form)
- `request.form.get()` — reads submitted field values
- `flash()` — stores a one-time message for the next page load
- `redirect(url_for("contact"))` — redirects after successful POST (prevents form resubmit on refresh)

---

### 🔹 5. Saving Messages to a JSON File

Submitted contact form data is saved to `messages.json` using `open()`:

```python
import json

def save_message(name, email, subject, message):
    messages = load_messages()           # Load existing list
    messages.append({                    # Add new entry
        "name": name,
        "email": email,
        "message": message,
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    })
    with open("messages.json", "w") as f:
        json.dump(messages, f, indent=2) # Save back to file
```

---

### 🔹 6. Flash Messages in Templates

Flask's `flash()` stores messages in the session. They appear once and disappear after display:

```html
<!-- In base.html -->
{% with messages = get_flashed_messages(with_categories=true) %}
  {% for category, message in messages %}
    <div class="flash flash--{{ category }}">
      {{ message }}
    </div>
  {% endfor %}
{% endwith %}
```

- `"success"` → green flash card
- `"error"` → red flash card

---

### 🔹 7. Static Files (CSS + JS)

Flask serves static files from the `static/` folder. `url_for('static', filename=...)` generates the correct URL:

```html
<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
<script src="{{ url_for('static', filename='js/main.js') }}"></script>
```

The CSS uses CSS custom properties (variables) for a consistent dark design system, and `@keyframes` animations for fade-up reveals.

---

## 🧠 Key Concepts Used

| Concept | Where Used |
|---|---|
| `@app.route()` | Defining URL endpoints for each page |
| `render_template()` | Rendering HTML files with Jinja2 |
| `request.method` | Detecting GET vs POST |
| `request.form.get()` | Reading contact form submissions |
| `flash()` | Success/error notification system |
| `redirect(url_for())` | Post-redirect-get pattern |
| Jinja2 `{% extends %}` | Template inheritance from `base.html` |
| Jinja2 `{% for %}` | Looping over projects, skills, education |
| `open()` + `json` | Saving contact messages to a file |
| CSS Grid + Flexbox | Responsive layout system |
| `IntersectionObserver` | Scroll-triggered animations in JS |

---

## 📤 Upload to GitHub

```bash
git init
git add .
git commit -m "Add Flask Portfolio Website"
git branch -M main
git remote add origin https://github.com/your-username/portfolio-flask.git
git push -u origin main
```

> 💡 Add `messages.json` to `.gitignore` to avoid pushing private contact data:
> ```
> echo "messages.json" >> .gitignore
> ```

---
