# 5.1 Frontend Snippets (Digital Planner)

Use these snippets and prompts when generating the Milestone 4 report content for section 5.1. Each snippet is ready for Codex with placement guidance.

## Snippet Placement

| Snippet ID | What it shows (conceptually) | Where to place in report | Prompt skeleton for Codex |
| --- | --- | --- | --- |
| F1 | Jinja/HTML template for the Digital Planner page (list of tasks + create form), matching the UI flow | After 5.1 first 1–2 paragraphs, captioned "Listing 5.1: Digital Planner Jinja Template" | Generate a Flask Jinja2 `planner.html` template that lists tasks from a `tasks` variable and includes a simple `<form>` to add a new task (title, due date, priority). Use semantic HTML5 and basic CSS classes, no frameworks. |
| F2 (optional) | Small JS function that calls the planner API (e.g., `/api/planner/tasks`) and updates the task list without page reload | Directly under F1, captioned "Listing 5.2: AJAX call to fetch planner tasks" | Write a vanilla JavaScript snippet that fetches JSON from `/api/planner/tasks` and repopulates a `<tbody id='task-body'>` with rows. Include basic error handling and `DOMContentLoaded` listener. |

## Concrete Skeletons

### F1 – `planner.html` (Jinja template skeleton)
```html
<!-- templates/planner.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Curtin Knowledge Hub – Digital Planner</title>
</head>
<body>
  <h1>Digital Planner</h1>

  <form method="post" action="{{ url_for('planner_bp.create_task') }}">
    <input type="text" name="title" placeholder="Task title" required>
    <input type="date" name="due_date" required>
    <select name="priority">
      <option value="low">Low</option>
      <option value="normal" selected>Normal</option>
      <option value="high">High</option>
    </select>
    <button type="submit">Add Task</button>
  </form>

  <table>
    <thead>
      <tr><th>Title</th><th>Due</th><th>Priority</th><th>Status</th></tr>
    </thead>
    <tbody id="task-body">
      {% for task in tasks %}
      <tr>
        <td>{{ task.title }}</td>
        <td>{{ task.due_date }}</td>
        <td>{{ task.priority }}</td>
        <td>{{ task.status }}</td>
      </tr>
      {% endfor %}
    </tbody>
  </table>

  <script src="{{ url_for('static', filename='planner.js') }}"></script>
</body>
</html>
```

Codex prompt: "Expand this skeleton into a clean, WCAG-friendly layout with basic CSS classes, but keep it framework-free (HTML/CSS/JS only)."

### F2 – `planner.js` (AJAX loader skeleton)
```javascript
// static/planner.js
document.addEventListener('DOMContentLoaded', () => {
  fetch('/api/planner/tasks')
    .then(res => res.json())
    .then(data => {
      const tbody = document.getElementById('task-body');
      tbody.innerHTML = '';
      data.forEach(task => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${task.title}</td>
          <td>${task.due_date}</td>
          <td>${task.priority}</td>
          <td>${task.status}</td>
        `;
        tbody.appendChild(row);
      });
    })
    .catch(err => {
      console.error('Failed to load tasks', err);
    });
});
```

Codex prompt: "Refine this JS to handle loading spinners, a basic retry on failure, and nicer error messages, still using vanilla JS."
