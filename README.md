# Obsidian Task Dashboard – Setup Guide

This is a guide for a simple setup for an Obsidian Task Dashboard.

The dashboard combines the `Tasks` plugin, `Meta Bind` plugin, JavaScript and custom CSS to provide a dynamic, Kanban-style overview of tasks throughout the vault.

The final JavaScript, Tasks queries and CSS files are maintained separately and can be linked from this document.

## 1. Requirements

### Required plugins

| Plugin        | Purpose                                                    | Creator | Required | Link |
| ------------- | ---------------------------------------------------------- | ------- | -------- | ---- |
| `Tasks`         | Task management, deadlines, priorities, status and queries | obsidian-tasks-group | Yes      | https://github.com/obsidian-tasks-group/obsidian-tasks |
| `Meta Bind`     | Interactive tag filters and dashboard buttons              | Moritz Jung | Yes      | https://github.com/mprojectscode/obsidian-meta-bind-plugin |
| `JS Engine`     | Executes JavaScript used by Meta Bind                      | Moritz Jung | Yes      | https://github.com/mprojectscode/obsidian-js-engine-plugin |

All required plugins are available through Obsidian's Community Plugins.
A separate Kanban plugin is not required. The Kanban-style board is created using Tasks queries and custom CSS.

### Required Obsidian configuration

1. Open Settings → Community plugins.
2. Enable community plugins.
3. Install and enable:
   - Tasks
   - Meta Bind
   - JS Engine
4. Restart Obsidian after installation.

## 2. Plugin configuration

### 2.1 Tasks

Open:

Settings → Community plugins → Tasks

Configure the following:

| Setting               | Configuration                                      |
| --------------------- | -------------------------------------------------- |
| Task format           | Tasks emoji format                                 |
| Global task filter    | Leave empty unless intentionally restricting tasks |
| JavaScript in queries | Enabled                                            |
| Task priorities       | Use Tasks' built-in priorities                     |
| Task dates            | Use Tasks' built-in date fields                    |

#### Enable JavaScript queries

The deadline overview uses JavaScript functions to assign tasks to deadline groups.
In the Tasks settings, locate the option controlling JavaScript execution in queries and enable it.
This is necessary because the dashboard uses functionality such as:

```
group by function
filter by function
```

Security: JavaScript in Tasks queries can access Obsidian and potentially local resources. Only execute query code you trust.
Documentation: [Tasks – JavaScript in Tasks Queries](https://publish.obsidian.md/tasks/Scripting/JavaScript+in+Tasks+Queries?utm_source=chatgpt.com)

### 2.2 Meta Bind

Open:
Settings → Community plugins → Meta Bind

| Setting                         | Configuration             |
| ------------------------------- | ------------------------- |
| Enable JavaScript               | ON                        |
| Enable syntax highlighting      | Optional                  |
| Enable editor right-click menu  | Optional                  |
| Excluded folders                | Templates (if applicable) |
| Date format                     | `YYYY-MM-DD`              |
| First weekday                   | Monday                    |
| Dev mode                        | OFF                       |
| Disable code block restrictions | OFF                       |

After enabling JavaScript, restart Obsidian.
Meta Bind provides three essential components:

- `meta-bind-js-view`: Generates the available tag filters.
- `multiSelect`: Allows selecting multiple tags.
- `meta-bind-button`: Provides Refresh tags and Reset filters actions.

Meta Bind's JavaScript execution must only be enabled for trusted code.

### 2.3 JS Engine

Install and enable JS Engine.
It provides the JavaScript execution environment required by the Meta Bind functionality used in our dashboard.
No additional configuration is required for the current implementation.
If Meta Bind reports a missing JavaScript dependency, verify that JS Engine is enabled and restart Obsidian.

## 3. Dashboard structure

Create a Markdown note named:

`Task Dashboard.md`

The dashboard consists of two sections:

1. Filters
2. Deadline Overview

A complete sample of the file can be copied from [sample_task_dashboard.md](sample_task_dashboard.md)

### 3.1 Frontmatter

Add the following properties:

```
---
cssclasses:
  - task-dashboard
focus_tags: []
tag_refresh: 0
---
```

| Property      | Purpose                                  |
| ------------- | ---------------------------------------- |
| `cssclasses`  | Activates dashboard-specific CSS         |
| `focus_tags`  | Stores the selected tag filters          |
| `tag_refresh` | Triggers recalculation of available tags |

The `focus_tags` property must be a list.
An empty list means that no tag filter is active.
The `tag_refresh` value acts as a change counter. It does not need to be reset to zero after refreshing.

### 3.2 Recommended note structure

```
---
cssclasses:
  - task-dashboard
focus_tags: []
tag_refresh: 0
---

# Task Dashboard

## Filters

<!-- Meta Bind button definitions / meta-bind-buttons -->

`BUTTON[refresh-tags,reset-filters]`

<!-- Dynamic tag filter / meta-bind-js-view -->

## Deadline Overview

<!-- Tasks query -->
```

The buttons and tag filter should be positioned above the deadline board.

## 4. Dynamic tag filtering

The dashboard automatically discovers the available tags from open tasks.

### 4.1 Tag discovery

The `meta-bind-js-view` script scans the vault and identifies incomplete tasks.
It collects:
- Tags directly assigned to incomplete tasks.
- Tags defined in the frontmatter of notes containing incomplete tasks.

The collected tags are:
1. Normalized.
2. Deduplicated.
3. Sorted alphabetically.
4. Converted into Meta Bind selection options.

For example:

```
#obsidian
#research
#simulation
#writing
```

The dashboard displays:

```
obsidian
research
simulation
writing
```

The `#` is removed only from the displayed label.
The actual tag value remains unchanged for filtering.

### 4.2 Selecting tags
The tag selector uses Meta Bind's `multiSelect`.
Selected tags are stored in:

```
focus_tags:
  - "#research"
  - "#simulation"
```

When no tags are selected:

```
focus_tags: []
```

All open tasks are displayed.
When one or more tags are selected, the dashboard displays matching tasks.
The current implementation uses OR logic: a task matching any selected tag is included.

### 4.3 Refresh tags

New tags may not immediately appear in the filter selector.
The Refresh tags button updates the `tag_refresh` property.
Its action uses:

```
action:
  type: updateMetadata
  bindTarget: tag_refresh
  evaluate: true
  value: "(Number(x) || 0) + 1"
```

The JavaScript view observes this property, so changing it triggers reevaluation.
The counter may increase indefinitely during normal use; resetting it is unnecessary.

### 4.4 Reset filters

The Reset filters button clears the selected tags.
Its action is:

```
action:
  type: updateMetadata
  bindTarget: focus_tags
  evaluate: true
  value: "[]"
```

The dashboard then displays all open tasks.

### 4.5 Inline buttons

Both buttons are positioned on the same line:

```
`BUTTON[refresh-tags,reset-filters]`
```

The corresponding button definitions must include:

```
id: refresh-tags
hidden: true
```

and:

```
id: reset-filters
hidden: true
```

This prevents the buttons from appearing twice.

## 5. Deadline Overview

The dashboard displays the open tasks in a two-row, four-column layout.
The cards will only appear, if existing tasks that match into the card are existing.

### 5.1 Deadline categories

| Category    | Definition                                      |
| ----------- | ----------------------------------------------- |
| Overdue     | Due date before today                           |
| Today       | Due today                                       |
| Tomorrow    | Due tomorrow                                    |
| This Week   | Due after tomorrow, within the current ISO week |
| Next Week   | Due within the following ISO week               |
| Next Month  | Due within the next calendar month              |
| Later       | Due beyond the defined periods                  |
| No Deadline | No due date assigned                            |

The grouping function must assign every task to exactly one category.

Important: The definition of the remaining days of the current month is separate from “Next Month.” Ensure that your linked final query specifies how those days are assigned. Do not assume the categories above are exhaustive without this rule.

### 5.2 Tasks query

The Tasks query is responsible for:

- Retrieving open tasks throughout the vault.
- Applying the selected tag filters.
- Grouping tasks by due date.
- Sorting tasks.
- Displaying task information and controls.

The query should use:

```
not done
```

The remainder of the query is maintained in the linked script.

## 6. Task cards

Each task is displayed as a compact card.

### Displayed information

| Element          | Purpose                          |
| ---------------- | -------------------------------- |
| Checkbox         | Complete the task                |
| Task description | Display the task                 |
| Priority         | Indicate importance              |
| Tags             | Show task categories             |
| Due date         | Display the deadline             |
| Source note link | Navigate to the originating note |
| Edit button      | Edit task details                |
| Postpone button  | Move the task to the next day    |

### Layout

The task description occupies the upper section of each card.
The source note link and action buttons are placed below the description to conserve horizontal space.
The footer elements are vertically centered so that links and icons align consistently.
The layout is controlled through the custom CSS file.

## 7. Custom CSS

Create a CSS snippet in:

```
<Vault>/.obsidian/snippets/task-dashboard.css
```

To activate it:

1. Open Settings → Appearance.
2. Scroll to CSS snippets.
3. Refresh the snippet list if necessary.
4. Enable `task-dashboard`.

The CSS is scoped to:

```
.task-dashboard
```

This prevents the dashboard styling from affecting other notes.

### CSS responsibilities

The stylesheet controls:
- Full-width dashboard layout.
- Four-column deadline board.
- Two-row arrangement.
- Task card appearance.
- Checkbox positioning.
- Footer alignment.
- Spacing and typography.
- Multi-column tag selection.
- Responsive behavior for narrower windows.

The tag selection grid uses a dedicated class:

```
.task-tag-grid
```

The Meta Bind input must include:

```
class(task-tag-grid)
```

The final CSS should be linked from this documentation.

## 8. Task creation conventions

Tasks can be created in any Markdown note included in the vault.

Example:

```
- [ ] Review simulation results #simulation 📅 2026-11-04
```

Or:

```
- [ ] Prepare research summary #research 🔼 📅 2026-11-06
```

The dashboard automatically discovers these tasks.

### Recommended conventions

| Attribute  | Convention                 |
| ---------- | -------------------------- |
| Task       | Standard Markdown checkbox |
| Category   | Obsidian tag               |
| Priority   | Tasks priority marker      |
| Deadline   | Tasks due-date marker      |
| Completion | Checkbox status            |
| Source     | Original Markdown note     |

A task's category can be inherited from the note's frontmatter tags, depending on the dashboard's filtering implementation.

For a predictable workflow, use consistent category names across the vault.

## 9. Verification checklist

After installation, verify the complete workflow.

###Setup verification

- [ ] Tasks, Meta Bind and JS Engine are installed and enabled.
- [ ] JavaScript execution is enabled in Tasks and Meta Bind.
- [ ] The dashboard contains the required frontmatter properties.
- [ ] The custom CSS snippet is enabled.
- [ ] Open tasks from different notes appear in the dashboard.
- [ ] Tasks are grouped into the correct deadline categories.
- [ ] Tag filters show only relevant categories and are sorted alphabetically.
- [ ] Selecting a tag filters the dashboard correctly.
- [ ] Reset filters clears all selections.
- [ ] Refresh tags discovers newly introduced task tags.
- [ ] Task checkboxes, editing and postponing work.
- [ ] Source links open the corresponding notes.

## 10. Linked implementation files

Maintain the scripts separately to simplify updates and troubleshooting.

| Component              | Reference            |
| ---------------------- | -------------------- |
| Dashboard note         | `[[Task Dashboard]]` |
| Tag discovery script   | Add link             |
| Tasks deadline query   | Add link             |
| Custom CSS             | Add link             |
| Dashboard setup manual | This document        |

### Final architecture

The solution deliberately separates responsibilities:

- Tasks manages the task data, deadlines, priorities and task operations.
- Meta Bind + JS Engine provides the interactive filters.
- Custom CSS provides the Kanban-style visualization.

No migration to another task-management plugin is needed. All tasks remain in their original Markdown notes, and the dashboard serves as a centralized view across the vault.

Maintenance: Keep the Tasks query, tag discovery script and CSS in your own linked implementation references. When modifying the dashboard, update the corresponding component without changing the others unnecessarily.