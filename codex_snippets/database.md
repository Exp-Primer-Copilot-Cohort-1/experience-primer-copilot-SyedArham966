# 5.3 Database Snippets (PostgreSQL + SQLAlchemy)

Use these database-oriented snippets and prompts for the Milestone 4 report section 5.3.

## Snippet Placement

| Snippet ID | What it shows (conceptually) | Where to place in report | Prompt skeleton for Codex |
| --- | --- | --- | --- |
| D1 | SQL schema for `planner_tasks` table (and FK to users) in PostgreSQL | First code in 5.3, captioned "Listing 5.5: PostgreSQL schema for planner tasks" | Generate a PostgreSQL `CREATE TABLE planner_tasks` statement with columns: `id SERIAL PRIMARY KEY`, `user_id INTEGER REFERENCES users(id) ON DELETE CASCADE`, `title VARCHAR(255) NOT NULL`, `description TEXT`, `due_date DATE`, `priority VARCHAR(20)`, `status VARCHAR(20) DEFAULT 'pending'`, `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`. Include an index on `(user_id, due_date)`. |
| D2 (optional) | DB access code in Python using SQLAlchemy ORM for querying tasks for a given user | Under D1, captioned "Listing 5.6: SQLAlchemy query for user tasks" | Write a SQLAlchemy ORM query function `get_user_tasks(user_id)` that returns planner tasks for a user ordered by `due_date` ascending, using a `PlannerTask` model with the fields from the SQL table above. |

## Concrete Skeletons

### D1 – PostgreSQL schema for planner tasks
```sql
-- schema/planner_tasks.sql
CREATE TABLE planner_tasks (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  due_date DATE NOT NULL,
  priority VARCHAR(20) NOT NULL DEFAULT 'normal',
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_planner_tasks_user_due
ON planner_tasks (user_id, due_date);
```

Codex prompt:

"Take this schema and add:
• a CHECK constraint that limits priority to ('low','normal','high'),
• a CHECK constraint that ensures status is one of ('pending','in_progress','done'),
• comments on the table and columns (PostgreSQL COMMENT ON), keeping it suitable for a planning-stage document."

### D2 – SQLAlchemy model + query helper
```python
# planner/models.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class PlannerTask(db.Model):
  __tablename__ = 'planner_tasks'

  id = db.Column(db.Integer, primary_key=True)
  user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
  title = db.Column(db.String(255), nullable=False)
  description = db.Column(db.Text)
  due_date = db.Column(db.Date, nullable=False)
  priority = db.Column(db.String(20), nullable=False, default='normal')
  status = db.Column(db.String(20), nullable=False, default='pending')
  created_at = db.Column(db.DateTime, server_default=db.func.now())

  def to_dict(self):
    return {
      "id": self.id,
      "title": self.title,
      "description": self.description,
      "due_date": self.due_date.isoformat() if self.due_date else None,
      "priority": self.priority,
      "status": self.status
    }

def get_user_tasks(user_id):
  return (PlannerTask.query
          .filter_by(user_id=user_id)
          .order_by(PlannerTask.due_date.asc())
          .all())
```

Codex prompt: "Polish this SQLAlchemy model and helper by adding type hints, docstrings explaining how it supports the Digital Planner feature, and an additional query method `get_overdue_tasks(user_id, today)`."
