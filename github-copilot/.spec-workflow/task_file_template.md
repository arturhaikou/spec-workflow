<task_file_template>
# Task [XXX]:[Task Name]

> **Applied Skill:** [Name of skill file] - [Brief note on what rule is being enforced]

## 1. Objective
[1-2 sentences explaining what this task achieves.]

## 2. File to Modify / Create
* **File Path:** `[Exact path from codebase root]`
* **Action:**[Create New File | Modify Existing File]

## 3. Code Implementation
**Imports Required:**
```[language][Exact import statements to add at the top of the file]
```

**Code to Add/Replace:**
* **Location:**[E.g., "Inside the `ValidateOrder` method, right after the null check"]
* **Snippet:**
```[language]
[Exact, copy-paste ready code]
```

## 4. Validation Steps
Execute the following commands to ensure this task was successful. Do NOT run the full application server.
```bash[e.g., dotnet build]
[e.g., dotnet test path/to/specific/test.cs]
```

</task_file_template>