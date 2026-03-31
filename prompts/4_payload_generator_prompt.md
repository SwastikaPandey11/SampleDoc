
---

# 📄 4. `payload_generator_prompt.md`

```md
# 📦 Task: Payload Generator

## Role: Senior Software Engineer

---

## 📥 Input

- Payload conditions
- Reference:
  - `EmtSpareRequest2737.java`

---

## 🎯 Goal

Generate request payload JSON.

---

## 🧠 Rules

- Follow exact structure from Java template
- Modify ONLY required fields
- Keep defaults unchanged

---

## ✍️ Examples

Condition → Payload:

- `force_flag = Y`
  → `"forceFlag": "Y"`

- `amount = 0`
  → `"amount": 0`

---

## 📤 Output Format

```json
{
  "field1": "value",
  "field2": 123
}