---
cssclasses:
  - task-dashboard
  - daily-task-note
focus_tags: []
tag_refresh: 0
---

# {{date:dddd, DD MMMM YYYY}}

### Filters
```meta-bind-button
id: refresh-tags
label: Refresh tags
style: default
hidden: true
actions:
  - type: updateMetadata
    bindTarget: tag_refresh
    evaluate: true
    value: "(Number(x) || 0) + 1"
```
```meta-bind-button
id: reset-filters
label: Reset filters
style: default
hidden: true
actions:
  - type: updateMetadata
    bindTarget: focus_tags
    evaluate: true
    value: "[]"
```
```meta-bind-button
id: reset-filters
label: Reset filters
style: default
hidden: true
actions:
  - type: updateMetadata
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

    const openTasks = cache.listItems.filter(item => item.task === " ");
    if (openTasks.length === 0) continue;

    const noteTags = cache.frontmatter?.tags ?? [];
    const normalizedTags = Array.isArray(noteTags)
        ? noteTags : String(noteTags).split(",");

    for (const tag of normalizedTags) {
        const clean = String(tag).trim().replace(/^#/, "").split(/\s+/)[0];
        if (clean) tags.add("#" + clean);
    }

    for (const item of openTasks) {
        const line = item.position.start.line;
        for (const entry of cache.tags ?? []) {
            if (entry.position.start.line === line) tags.add(entry.tag);
        }
    }
}

const sorted = [...tags].sort((a, b) =>
    a.replace(/^#/, "").localeCompare(b.replace(/^#/, ""), undefined, { sensitivity: "base" })
);

if (sorted.length === 0) return "No tags found on open tasks.";

const options = sorted.map(tag => `option(${tag}, ${tag.replace(/^#/, "")})`).join(", ");
return engine.markdown.create(
    "```meta-bind\n" +
    `INPUT[multiSelect(${options}, class(task-tag-grid)):focus_tags]\n` +
    "```"
);
```
## Today's Tasks
```tasks
not done
hide toolbar

filter by function \
const day = moment('{{date:YYYY-MM-DD}}'); \
const due = task.due.moment; \
const scheduled = task.scheduled.moment; \
if (due?.isBefore(day, 'day')) return false; \
return (due?.isSame(day, 'day') || false) || (scheduled?.isSame(day, 'day') || false);

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
const p = task.priorityNumber; \
if (p <= 1) return '%%1%% Highest / High'; \
if (p === 2) return '%%2%% Medium'; \
if (p === 3) return '%%3%% Normal'; \
return '%%4%% Low / Lowest';

sort by priority
sort by due
show group count
hide task count
show backlink
hide created date
hide start date
hide scheduled date
```
### Quick Capture
- 

## Next Tasks
```tasks
not done
hide toolbar

filter by function \
const day = moment('{{date:YYYY-MM-DD}}'); \
const due = task.due.moment; \
const scheduled = task.scheduled.moment; \
const start = task.start.moment; \
if (!due || !due.isAfter(day, 'day')) return false; \
if (scheduled?.isSame(day, 'day')) return false; \
const tomorrow = day.clone().add(1, 'day'); \
const friday = day.isoWeekday() === 5; \
const horizon = friday ? day.clone().add(1, 'week').endOf('isoWeek') : tomorrow; \
const upcoming = due.isBetween(tomorrow, horizon, 'day', '[]'); \
const actionable = start?.isSameOrBefore(day, 'day') || false; \
return upcoming || actionable;

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
const p = task.priorityNumber; \
if (p <= 1) return '%%1%% Highest / High'; \
if (p === 2) return '%%2%% Medium'; \
if (p === 3) return '%%3%% Normal'; \
return '%%4%% Low / Lowest';

sort by priority
sort by due
show group count
hide task count
show backlink
hide created date
hide start date
hide scheduled date
```
### Quick Capture
- 

-----
## Notes
- 

## Links
- 

