---
date: <% tp.date.weekday("YYYY-MM-DD", 0) %>
type: monthly
tags: "#moc #daily/<% tp.date.now("YYYY") %>/<% tp.date.now("MM") %> "
description: Monthly note for <% tp.date.now("MMMM") %> <% tp.date.now("YYYY")%> 
---
# <% tp.date.now("MMMM") %>  <% tp.date.now("YYYY") %>  

## Overview

Provide a brief summary of the month's work, highlighting key achievements and significant milestones.

---
## Project Updates

### Project 1: [Project Name]

**Progress:**
- [Detail the progress made on the project]

**Next Steps:**
- [List the next steps or upcoming tasks]

**Issues:**
- [Describe any issues or blockers encountered]

### Project 2: [Project Name]

**Progress:**
- [Detail the progress made on the project]

**Next Steps:**
- [List the next steps or upcoming tasks]

**Issues:**
- [Describe any issues or blockers encountered]

---
## Meetings 

```dataview  
TABLE attendees As Attendees, description As Description
From -"3_Resources/Obsidian/Templates" 
WHERE  date.year = this.date.year
WHERE  date.month = this.date.month

WHERE contains(type,"meeting")
Sort file.name ASC
Limit 7  
```

---
## Upcoming Goals and Plans for Next Month

- [Goal 1]
- [Goal 2]
- [Goal 3]

---
## Additional Notes

- [Any additional information or notes]

---
## Weekly Breakdown
```dataviewjs
const tracked = {};
const tagsRegex = /#\S+(?:\s\S+)*/g;

// Define the start and end dates for the desired month
const monthStartDate = moment(dv.current().file.frontmatter.date).startOf('month');
const monthEndDate = moment(dv.current().file.frontmatter.date).endOf('month');

// Gather data
dv.pages('"0_daily"').file.lists
  .where(x => x.section.subpath === "Work log" && !x.text.includes("#sick_day") && 
!x.text.includes("#vacation_day")).array()
  .forEach(x => {
    const times = x.text.match(/^(\d{2}:\d{2})-(\d{2}:\d{2})/);
    if (times) {
      const start = moment(times[1], 'HH:mm');
      const end = moment(times[2], 'HH:mm');
      const minutes = moment.duration(end.diff(start)).asMinutes();
      const date = x.path.match(/(\d{4}-\d{2}-\d{2})/)[1];

      if (moment(date).isBetween(monthStartDate, monthEndDate, null, '[]')) {
        const weekOfMonth = moment(date).week() - moment(date).startOf('month').week() + 1;
        const monthName = moment(date).format('MMMM');
        const weekStart = moment(date).startOf('week').format('YYYY-MM-DD');
        const weekEnd = moment(date).endOf('week').format('YYYY-MM-DD');
        const week = `[${monthName}, Week ${weekOfMonth}](${weekStart}_to_${weekEnd})`;

        const tags = x.text.match(tagsRegex)?.map(tag => tag.trim());
        const description = x.text.replace(/^\d{2}:\d{2}-\d{2}:\d{2}\s+/, '');

        if (!tracked[week]) tracked[week] = {};
        if (tracked[week][date]) {
          tracked[week][date].entries.push({
            path: x.path,
            start: start.format('HH:mm'),
            end: end.format('HH:mm'),
            minutes: minutes,
            tags: tags || [],
            description: description || ''
          });
          tracked[week][date].minutes += minutes;
          tracked[week][date].tags = [...new Set(tracked[week][date].tags.concat(tags || []))];
        } else {
          tracked[week][date] = {
            path: x.path,
            entries: [{
              path: x.path,
              start: start.format('HH:mm'),
              end: end.format('HH:mm'),
              minutes: minutes,
              tags: tags || [],
              description: description || ''
            }],
            minutes: minutes,
            tags: tags || [],
            description: description || ''
          };
        }
      }
    }
  });

const tables = {};

// Iterate through tracked entries to fill the tables
Object.keys(tracked).sort((a, b) => a.localeCompare(b))
  .forEach(week => {
    const weekEntries = tracked[week];
    const table = [];

    const sortedDates = Object.keys(weekEntries).sort((a, b) => a.localeCompare(b));

    sortedDates.forEach(date => {
      const dailyEntry = weekEntries[date];
      const link = `[[${dailyEntry.path}#Work log|${moment(date).format('dddd D MMMM')}]]`;

      table.push(['**' + link + '**', '**' + (dailyEntry.minutes / 60).toFixed(1) + '**', '']);

      dailyEntry.entries.sort((a, b) => moment(a.start, 'HH:mm').diff(moment(b.start, 'HH:mm'))).forEach(entry => {
        table.push([
          '',
          (entry.minutes / 60).toFixed(1),
          entry.description
        ]);
      });
    });

    tables[week] = table;
  });

// Render tables for each week
Object.keys(tables).forEach(week => {
  dv.header(3, week);
  dv.table(['Date', 'Hours', 'Description'], tables[week]); // Include 'Date', 'Hours', and 'Description' in the table header
});

```




<%*tp.file.rename(tp.date.now("YYYY")+"_"+tp.date.now("MM") );%>


<% tp.user.pin_me() %>