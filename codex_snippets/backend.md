# 5.2 Backend Snippets (Flask Planner Blueprint)

Use these snippets and prompts when documenting the backend portion of the Digital Planner in section 5.2.

## Snippet Placement

| Snippet ID | What it shows (conceptually) | Where to place in report | Prompt skeleton for Codex |
| --- | --- | --- | --- |
| B1 | Core Flask blueprint for the planner: `/planner` (HTML) + `/api/planner/tasks` (JSON GET) + `/api/planner/tasks` (POST) | Right after your 5.2 architecture description, captioned "Listing 5.3: Flask planner blueprint" | Create a Flask blueprint called `planner_bp` with three routes: `/planner` (renders `planner.html` with tasks from the DB), `/api/planner/tasks` (GET returns list of tasks as JSON), and `/api/planner/tasks` (POST creates a new task). Use SQLAlchemy models `User` and `PlannerTask` and return proper status codes. |
| B2 (optional) | Service/helper function that encapsulates business logic for creating a planner task | Under B1, captioned "Listing 5.4: Planner service function" | Write a Python function `create_planner_task(user_id, title, due_date, priority)` that validates inputs, creates a `PlannerTask` SQLAlchemy object, commits to the DB, and returns it. Raise `ValueError` for invalid input. |

## Concrete Skeletons

### B1 – `planner_bp` blueprint skeleton
```python
# planner/routes.py
from flask import Blueprint, render_template, request, jsonify
from .models import PlannerTask, db
from flask_login import current_user, login_required

planner_bp = Blueprint('planner_bp', __name__)

@planner_bp.route('/planner')
@login_required
def planner_page():
  tasks = PlannerTask.query.filter_by(user_id=current_user.id).order_by(PlannerTask.due_date).all()
  return render_template('planner.html', tasks=tasks)

@planner_bp.route('/api/planner/tasks', methods=['GET'])
@login_required
def get_tasks():
  tasks = PlannerTask.query.filter_by(user_id=current_user.id).order_by(PlannerTask.due_date).all()
  return jsonify([t.to_dict() for t in tasks]), 200

@planner_bp.route('/api/planner/tasks', methods=['POST'])
@login_required
def create_task_route():
  data = request.get_json() or request.form
  task = create_planner_task(
      user_id=current_user.id,
      title=data.get('title'),
      due_date=data.get('due_date'),
      priority=data.get('priority', 'normal')
  )
  return jsonify(task.to_dict()), 201
```

Codex prompt:

"Take this blueprint skeleton and:
• add proper error handling (400 for bad input, 500 for unexpected errors),
• validate required fields,
• ensure it integrates with SQLAlchemy and a PlannerTask.to_dict() method,
• keep it readable for inclusion in a university report."

### B2 – Service function `create_planner_task`
```python
# planner/services.py
from datetime import date
from .models import PlannerTask, db

def create_planner_task(user_id, title, due_date, priority='normal'):
  if not title:
    raise ValueError("Title is required")
  # Convert due_date string "YYYY-MM-DD" to date object
  if isinstance(due_date, str):
    due_date = date.fromisoformat(due_date)

  task = PlannerTask(
      user_id=user_id,
      title=title.strip(),
      due_date=due_date,
      priority=priority,
      status='pending'
  )
  db.session.add(task)
  db.session.commit()
  return task
```

Codex prompt: "Harden this function: add more validation (max title length, valid priority values), wrap DB operations in try/except, and log errors using Python’s logging module."
