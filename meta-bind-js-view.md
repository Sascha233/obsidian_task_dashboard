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