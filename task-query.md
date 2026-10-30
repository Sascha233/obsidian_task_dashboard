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