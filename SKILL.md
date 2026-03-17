To make this **skill.md** truly functional for an AI collaborator, we need to define the "Transformation Logic." This section instructs the AI on how to translate natural language requirements into precise DBML structures.

Here is the updated **skill.md** including the **Operational Logic** section.

---

# DBML Skill Guide & Operational Protocol

## 1. Core Syntax Reference
*(Abbreviated for brevity—see previous section for full details on Tables, Enums, and Partials)*

* **Tables**: `Table name { column type [settings] }`
* **Relationships**: `Ref: table1.col > table2.col` (Use `>` for many-to-one, `-` for one-to-one).
* **Enums**: `enum name { value1 \n value2 }`
* **Partials**: `TablePartial name { ... }` injected via `*name`.

---

## 2. AI Operational Logic: English → DBML
When receiving natural language instructions, the AI should apply the following mapping rules:

### A. Entity Extraction
* **Requirement:** "We need to track users and their posts."
* **Action:** Create two `Table` blocks: `users` and `posts`.
* **Naming Convention:** Use `snake_case` for table and column names.

### B. Relationship Mapping
| If the user says... | AI should type... |
| :--- | :--- |
| "Users can have many posts" | `Ref: posts.user_id > users.id` |
| "Each profile belongs to exactly one user" | `Ref: profiles.user_id - users.id` |
| "A student can take many courses, and a course has many students" | Create a join table and use `<>` or two `>` refs. |
| "Automatically delete posts when a user is removed" | Add `[delete: cascade]` to the `Ref`. |

### C. Constraint Mapping
* **Requirement:** "The email must be unique and always provided."
* **Action:** `email varchar [unique, not null]`
* **Requirement:** "Set the sign-up date to the current time automatically."
* **Action:** `created_at timestamp [default: `now()`]`

### D. Reusability Logic (Partials)
* **Requirement:** "All my tables need 'created_at' and 'updated_at' fields."
* **Action:** Define a `TablePartial timestamps { ... }` and inject `*timestamps` into every table definition.

---

## 3. Transformation Examples

### Scenario 1: Simple Blog Requirements
**User:** "I want a system where users have a bio and can write multiple posts. Posts should have a title and content."

**AI Response (DBML):**
```dbml
Table users {
  id integer [pk]
  username varchar
  bio text
}

Table posts {
  id integer [pk]
  title varchar
  content text
  user_id integer [ref: > users.id]
}
```

### Scenario 2: Optimization & Cleanup
**User:** "Make sure the status of an order can only be 'pending', 'shipped', or 'delivered'."

**AI Response (DBML):**
```dbml
enum order_status {
  pending
  shipped
  delivered
}

Table orders {
  id integer [pk]
  status order_status
}
```

---

## 4. Best Practices for the AI
1.  **Always** include a `primary key` (usually `id`) unless specified otherwise.
2.  **Group** all `Ref` definitions at the bottom of the file for better readability unless requested inline.
3.  **Default** to `integer` for ID columns and `varchar` for short strings.
4.  **Comment** complex logic using `//` or the `note:` attribute.

---