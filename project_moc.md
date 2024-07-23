---
title: <% tp.file.title %>
date: <% tp.file.creation_date("YYYY-MM-DD") %>
type: MOC
tags: "   "
description: "MOC of the ____ Project."
---

----


## Meetings

```dataview
TABLE attendees, description
FROM "2_Areas/PostDoc/meetings" AND #ualberta/postdoc 
SORT file.ctime DESC

```

---

## Notes
```dataview
Table without ID file.link as "Title", description, file.mday as "Date Modified"
FROM #ualberta/postdoc AND -"2_Areas/PostDoc/meetings"
SORT file.cday DESC
```

---

## Tasks

```tasks
filter by function task.status.symbol == ' '
tags include #climatedownscaling 

```



