
---

# 📄 2. `segregation_prompt.md`

```md
# 🔍 Task: Condition Segregation

## Role: Senior Software Engineer

You are given:
- Column F (Conditions)
- Column G (DB_Table)

---

## 🎯 Goal

Split conditions into:

1. 🗄️ DB Operations
2. 📦 Payload Modifications

---

## 🧠 Rules

### DB Condition Indicators:
- Mentions table/column updates
- Words like:
  - update
  - insert
  - delete
  - exists in DB
  - record should be present

👉 These belong to DB Query

---

### Payload Condition Indicators:
- Input values
- Flags
- Request parameters

👉 These belong to Payload

---

## 📤 Output Format

```json
{
  "db_conditions": ["..."],
  "payload_conditions": ["..."]
}