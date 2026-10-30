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