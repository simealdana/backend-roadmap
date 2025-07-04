# Final Assignment – Build a RESTful API for a TODO App

## Objective
Design and implement an in-memory RESTful API (no database) to manage personal tasks (To-Do list). The API must support full CRUD operations and include additional features like filtering and a bonus challenge.

---

##  Main Resource: `tasks`

Each task should have the following structure:

```json
{
  "id": 1,
  "title": "Buy milk",
  "description": "Go to the store and buy skimmed milk",
  "completed": false,
  "priority": "high", // possible values: low, medium, high
  "createdAt": "2025-07-04T12:30:00Z",
  "dueDate": "2025-07-05T23:59:00Z",
  "tags": ["shopping", "urgent"]
}
```

---

##  Required Endpoints

Implement the following REST endpoints using proper conventions:

| Method | Route          | Description                                          |
|--------|----------------|------------------------------------------------------|
| GET    | `/tasks`       | List all tasks. Support filters.                    |
| GET    | `/tasks/:id`   | Get details of a task by ID.                        |
| POST   | `/tasks`       | Create a new task.                                  |
| PUT    | `/tasks/:id`   | Fully replace an existing task.                     |
| PATCH  | `/tasks/:id`   | Partially update a task (e.g., mark as completed).  |
| DELETE | `/tasks/:id`   | Delete a task by ID.                                |

---

##  Filters in `GET /tasks`

Allow filtering by one or more of the following query parameters:

- `completed` (e.g.: `?completed=true`)
- `priority` (e.g.: `?priority=high`)
- `tags` (e.g.: `?tags=urgent`) → filters tasks that include that tag
- `overdue=true` → returns tasks whose `dueDate` is earlier than the current date

**Example:**  
`GET /tasks?completed=false&priority=high&tags=work`

---

##  Technical Requirements

- The API must work **without a database**. Data can be stored in an in-memory array.
- Response format should always be **JSON**.
- Each response should include an appropriate **HTTP status code** (200, 201, 204, 400, 404, etc.).
- Validate required fields on `POST` and `PUT`, returning 400 if there are errors.
- The API should be easily testable using **Postman** or similar tools.

---

##  Bonus Challenge (Optional)

Add a new field: `locked: boolean`.  
If a task has `locked: true`, it **cannot be modified or deleted** (via PUT, PATCH, or DELETE).  
The API must return a 403 error with a clear message like `"This task is locked and cannot be modified."`

---

##  Deliverable

- Source code in a repository or zip file.
- A README with instructions on how to run the API and usage examples.
- (Optional) A Postman collection with tested requests.

---

##  Evaluation Criteria

- Correct use of REST principles and HTTP verbs.
- Quality of the data model and validations.
- Proper error handling and use of HTTP status codes.
- Ability to extend logic with filters and business rules (e.g., locked tasks).
