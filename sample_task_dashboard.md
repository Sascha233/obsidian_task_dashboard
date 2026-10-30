```
---
cssclasses:
  - task-dashboard
focus_tags: []
tag_refresh: 0
---
```

# Task Dashboard

## Filters
```meta-bind-button
id: refresh-tags
label: Refresh tags
style: default
hidden: true
action:
  type: updateMetadata
  bindTarget: tag_refresh
  evaluate: true
  value: "(Number(x) || 0) + 1"
```
```meta-bind-button
id: reset-filters
label: Reset filters
style: default
hidden: true
action:
  type: updateMetadata
  bindTarget: focus_tags
  evaluate: true
  value: "[]"
```
`BUTTON[refresh-tags,reset-filters]`
```meta-bind-js-view
{focus_tags} as selected
{tag_refresh} as refresh
---
const tags = new Set();

for (const file of app.vault.getMarkdownFiles()) {
    const cache = app.metadataCache.getFileCache(file);

    if (!cache?.listItems) continue;

    // Find incomplete tasks
    const openTasks = cache.listItems.filter(
        item => item.task === " "
    );

    if (openTasks.length === 0) continue;

    // Collect tags from the note's properties
    const noteTags = cache.frontmatter?.tags ?? [];
    const normalizedTags = Array.isArray(noteTags)
        ? noteTags
        : String(noteTags).split(",");

    for (const tag of normalizedTags) {
	    const clean = String(tag)
	        .trim()
	        .replace(/^#/, "")
	        .split(/\s+/)[0];
	
	    if (clean) tags.add("#" + clean);
	}

    // Collect tags directly from incomplete task lines
    for (const item of openTasks) {
        const line = item.position.start.line;

        for (const entry of cache.tags ?? []) {
            if (entry.position.start.line === line) {
                tags.add(entry.tag);
            }
        }
    }
}

// Sort alphabetically
const sorted = [...tags].sort((a, b) =>
    a.replace(/^#/, '').localeCompare(
        b.replace(/^#/, ''),
        undefined,
        { sensitivity: 'base' }
    )
);

if (sorted.length === 0) {
    return "No tags found on open tasks.";
}

// Generate Meta Bind options
const options = sorted
    .map(tag => `option(${tag}, ${tag.replace(/^#/, '')})`)
    .join(", ");

// Render the selection field
return engine.markdown.create(
    "```meta-bind\n" +
    `INPUT[multiSelect(${options}, class(task-tag-grid)):focus_tags]\n` +
    "```"
);
```
## Deadline Overview

```tasks
hide toolbar
not done

filter by function \
const selected = query.file.property('focus_tags'); \
if (!Array.isArray(selected) || selected.length === 0) return true; \
const normalize = value => String(value).trim().replaceAll('"', '').replaceAll("'", '').replace('#', '').split(' ')[0]; \
const selectedTags = selected.map(normalize); \
const directTags = (task.tags || []).map(normalize); \
const noteTags = task.file.property('tags') || []; \
const inheritedTags = (Array.isArray(noteTags) ? noteTags : String(noteTags).split(',')).map(normalize); \
return selectedTags.some(tag => directTags.includes(tag) || inheritedTags.includes(tag));

group by function \
const date = task.due.moment; \
const now = moment(); \
const tomorrow = now.clone().add(1, 'day'); \
const endOfWeek = now.clone().endOf('isoWeek'); \
const nextWeekStart = now.clone().add(1, 'week').startOf('isoWeek'); \
const nextWeekEnd = nextWeekStart.clone().endOf('isoWeek'); \
const nextMonthStart = now.clone().add(1, 'month').startOf('month'); \
const nextMonthEnd = nextMonthStart.clone().endOf('month'); \
const label = (order, name) => `%%${order}%% ${name}`; \
if (!date) return label(8, 'No Deadline'); \
if (!date.isValid()) return label(9, 'Invalid Date'); \
if (date.isBefore(now, 'day')) return label(1, 'Overdue'); \
if (date.isSame(now, 'day')) return label(2, 'Today'); \
if (date.isSame(tomorrow, 'day')) return label(3, 'Tomorrow'); \
if (date.isSameOrBefore(endOfWeek, 'day')) return label(4, 'This Week'); \
if (date.isBetween(nextWeekStart, nextWeekEnd, 'day', '[]')) return label(5, 'Next Week'); \
if (date.isBetween(nextMonthStart, nextMonthEnd, 'day', '[]')) return label(6, 'Next Month'); \
return label(7, 'Later');

sort by priority
sort by due

show group count
hide task count
show backlink
hide created date
hide start date
hide scheduled date
```