---
date: {{date}}
type: daily
tags: "#daily/{{date:YYYY}}/{{date:MM}}"
description: "Daily note for {{date}}"
---
[[<% tp.date.yesterday() %>|yesterday]]
[[<% tp.date.tomorrow() %>|tomorrow]]
[[<% tp.date.weekday("YYYY-MM-DD", 0) %>_to_<% tp.date.weekday("YYYY-MM-DD", 6) %>|this week]]
[[0_daily/training/<% tp.date.weekday("YYYY-MM-DD", 0) %>_to_<% tp.date.weekday("YYYY-MM-DD", 6) %>|training log]]

---
## Training log



---
## Personal log

<% tp.user.pin_me() %>

---
## Work log
- 


---
## Meetings
```dataview  
TABLE description As Description 
From -"3_resources/obsidian/templates" 
WHERE  date = striptime(this.date)
WHERE contains(type,"meeting")
Sort file.name ASC
Limit 7  
```

---
## Tasks
```tasks
status.type is not CANCELLED

filter by function task.done.moment?.isSame(moment("{{date}}"), 'day') || (task.scheduled.moment?.isSame(moment("{{date}}"), 'day') && !task.done.moment)||(task.scheduled.moment?.isBefore(moment("{{date}}"), 'day') && !task.done.moment)|| false

sort by status

sort by description

```

---
## Scratch