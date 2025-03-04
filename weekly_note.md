---
date: <% tp.date.weekday("YYYY-MM-DD", 0) %>
type: weekly
tags: "#moc #daily/<% tp.date.now("YYYY") %>/<% tp.date.now("MM") %> "
description: Weekly note from <% tp.date.weekday("YYYY-MM-DD", 0)%> to <% tp.date.weekday("YYYY-MM-DD", 6)%>
---
[[<% tp.date.weekday("YYYY-MM-DD", -7) %>_to_<% tp.date.weekday("YYYY-MM-DD", -1) %>|last week]]
[[<% tp.date.weekday("YYYY-MM-DD", 7) %>_to_<% tp.date.weekday("YYYY-MM-DD", 13) %>|next week]]
[[0_daily/training/<% tp.date.weekday("YYYY-MM-DD", 0) %>_to_<% tp.date.weekday("YYYY-MM-DD", 6) %>|training log]]

##  [[time_tracking_postdoc|Work log]]

```dataviewjs
const tracked = {};
const tagsRegex = /#\S+(?:,\s*#\S+)*/g;

// Define the start date for the desired week
const weekStartDate = dv.current().file.frontmatter.date;

// Define the end date for the desired week by adding 6 days to weekStartDate
const weekEndDate = moment(weekStartDate).add(6, 'days').format('YYYY-MM-DD');

// Gather data
dv.pages('"0_daily"').file.lists
  .where(x => x.section.subpath === "Work log" && !x.text.includes("#sick_day") && 
  !x.text.includes("#vacation_day")).array() // Add condition to exclude entries with tags "#sick_day" or "vacation_day"
  .forEach(x => {
    // Find the start/end times for each bullet point
    const times = x.text.match(/^(\d{2}:\d{2})-(\d{2}:\d{2})/)
    if (times) {
      const start = moment(times[1], 'HH:mm');
      const end = moment(times[2], 'HH:mm');
      const minutes = moment.duration(end.diff(start)).asMinutes();
      const dateMatch = x.path.match(/(\d{4}-\d{2}-\d{2})/);
      const date = dateMatch ? moment(dateMatch[1], 'YYYY-MM-DD') : null;
      if (!date) return; // Skip if date parsing fails

      const weekStart = date.clone().startOf('week').format('YYYY-MM-DD');
      const weekEnd = date.clone().endOf('week').format('YYYY-MM-DD');
      const week = `[[${weekStart}_to_${weekEnd}]]`; // Enclosing the week in double square brackets

      // Check if the date falls within the specified week start and end dates
      if (date.isBetween(weekStartDate, weekEndDate, null, '[]')) {
        // Extract tags from the note
        const tags = x.text.match(tagsRegex)?.map(tag => tag.trim()); // Extracts all hashtags from the note
        const description = x.text.replace(/^(\d{2}:\d{2}-\d{2}:\d{2}\s)/, ''); // Extracting text after time values

        if (!tracked[week]) tracked[week] = {};
        if (tracked[week][date]) {
          tracked[week][date].entries.push({
            path: x.path,
            start: start.format('HH:mm'),
            end: end.format('HH:mm'),
            minutes: minutes,
            tags: tags || [],
            description: description || '' // Initialize description as an empty string
          });
          tracked[week][date].minutes += minutes;
          // Append tags to the existing tracked entry
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
              description: description || '' // Initialize description as an empty string
            }],
            minutes: minutes,
            tags: tags || [],
            description: description || '' // Initialize description as an empty string
          };
        }
      }
    }
  });

const hours = minutes => (minutes / 60).toFixed(1);

const table = [];
let totalWeekTime = 0;
let totalPostdocAdminTime = 0;
// let totalTeachingTime = 0;
let totalPIWOTime = 0;
let totalMixedwoodTime = 0; // Updated variable name
let totalCAWATime = 0;
let totalgeospatial_catalogTime = 0;

// Push daily values and accumulate totals
Object.keys(tracked)
  .sort((a, b) => b.localeCompare(a))
  .forEach(week => {
    Object.keys(tracked[week])
      .sort((a, b) => b.localeCompare(a))
      .forEach(date => {
        const dailyEntry = tracked[week][date];
        const dailyTotal = dailyEntry.entries.reduce((acc, curr) => acc + curr.minutes, 0);
        totalWeekTime += dailyTotal;
        totalPostdocAdminTime += getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/admin');
        // totalTeachingTime += getTaggedMinutes(dailyEntry.entries, '#ualberta/teaching/BIOL_471_571');
        totalPIWOTime += getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/PIWO');
        totalMixedwoodTime += getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/mixedwoods'); // Updated variable name
        totalCAWATime += getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/CAWA');
        totalgeospatial_catalogTime += getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/geospatial_catalog');

        // Push daily summary row with date linking to the file
        const link = `[[${dailyEntry.path}#Work log|${moment(date).format('dddd D MMMM')}]]`;
        table.push([
          link, // Enclose date in double square brackets and link back to source file

          hours(dailyTotal),
          hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/admin')),
          //hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/teaching/BIOL_471_571')),
          hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/PIWO')),
          hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/mixedwoods')), // Updated column header
          hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/CAWA')),
           hours(getTaggedMinutes(dailyEntry.entries, '#ualberta/postdoc/geospatial_catalog')),
        ]);
      });
  });

// Sort table in descending order by date
table.sort((a, b) => {
  const dateA = moment(a[0].match(/\d{4}-\d{2}-\d{2}/)[0], 'YYYY-MM-DD');
  const dateB = moment(b[0].match(/\d{4}-\d{2}-\d{2}/)[0], 'YYYY-MM-DD');
  return dateB - dateA;
});

// Push total row at the top
table.unshift([
  'Total',
  hours(totalWeekTime),
  hours(totalPostdocAdminTime),
  // hours(totalTeachingTime),
  hours(totalPIWOTime),
  hours(totalMixedwoodTime), // Updated column header
  hours(totalCAWATime),
  hours(totalgeospatial_catalogTime)
]);

function getTaggedMinutes(entries, tag) {
  return entries.reduce((total, entry) => {
    if (entry.tags.includes(tag)) {
      total += entry.minutes;
    }
    return total;
  }, 0);
}

dv.table(['Day/Week', 'Total', 'Admin', 'PIWO', 'Mixedwood', 'CAWA', 'Geospatial'], table); // Updated column header

```



```dataviewjs
const tracked = {};
const tagsRegex = /#\S+(?:\s\S+)*/g;

// Define the start and end dates for the desired week
const weekStartDate = dv.current().file.frontmatter.date;
const weekEndDate = moment(weekStartDate).add(6, 'days').format('YYYY-MM-DD');

// Gather data
dv.pages('"0_daily"').file.lists
  .where(x => x.section.subpath === "Work log" && !x.text.includes("#sick_day") && 
!x.text.includes("#vacation_day")).array() // Add condition to exclude entries with tags "#sick_day"  or "vacation_day"
  .forEach(x => {
    // Find the start/end times for each bullet point
    const times = x.text.match(/^(\d{2}:\d{2})-(\d{2}:\d{2})/)
    if (times) {
      const start = moment(times[1], 'HH:mm');
      const end = moment(times[2], 'HH:mm');
      const minutes = moment.duration(end.diff(start)).asMinutes();
      const date = x.path.match(/(\d{4}-\d{2}-\d{2})/)[1];
      
      // Check if the date falls within the specified week start and end dates
      if (moment(date).isBetween(weekStartDate, weekEndDate, null, '[]')) {
        const week = `[[${weekStartDate}_to_${weekEndDate}]]`;
        // Extract tags from the note
        const tags = x.text.match(tagsRegex)?.map(tag => tag.trim()); // Extracts all hashtags from the note
        const description = x.text.replace(/^(\d{2}:\d{2}-\d{2}:\d{2}\s)/, ''); // Extracting text after time values

        if (!tracked[week]) tracked[week] = {};
        if (tracked[week][date]) {
          tracked[week][date].entries.push({
            path: x.path,
            start: start.format('HH:mm'),
            end: end.format('HH:mm'),
            minutes: minutes,
            tags: tags || [],
            description: description || '' // Initialize description as an empty string
          });
          tracked[week][date].minutes += minutes;
          // Append tags to the existing tracked entry
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
              description: description || '' // Initialize description as an empty string
            }],
            minutes: minutes,
            tags: tags || [],
            description: description || '' // Initialize description as an empty string
          };
        }
      }
    }
  });

const hours = minutes => (minutes / 60).toFixed(1);

const table = [];
let totalWeekMinutes = 0;

// Calculate total hours before iterating through daily entries
Object.keys(tracked).sort((a, b) => b.localeCompare(a))
  .forEach(weekDate => {
    const weekEntries = tracked[weekDate];

    Object.keys(weekEntries).forEach(date => {
      totalWeekMinutes += weekEntries[date].minutes;
    });
  });

// Push total hours row to the top of the table
table.push(['**Total Hours**', '**' + hours(totalWeekMinutes) + '**', '']);
// Horizontal line after the total hours row
//table.push(['----', '----', '----']);

// Iterate through tracked entries to fill the table
Object.keys(tracked).sort((a, b) => b.localeCompare(a))
  .forEach(weekDate => {
    const weekEntries = tracked[weekDate];

    // Sort daily values within the week summary in descending order by date
    const sortedDates = Object.keys(weekEntries).sort((a, b) => b.localeCompare(a));

    sortedDates.forEach(date => {
      const dailyEntry = weekEntries[date];
      const link = `[[${dailyEntry.path}#Work log|${moment(date).format('dddd D MMMM')}]]`;

      // Push daily summary above the entries for each day
      table.push(['**' + link + '**', '**' + hours(dailyEntry.minutes) + '**', '']);

      // Push individual time entries (sorted in descending order)
      dailyEntry.entries.sort((a, b) => moment(b.start, 'HH:mm').diff(moment(a.start, 'HH:mm'))).forEach(entry => {
        table.push([
          `${entry.start}-${entry.end}`,
          hours(entry.minutes),
          entry.description
        ]);
      });
    });
  });

dv.table(['Date', 'Hours', 'Description'], table);


```

---
## [[meetings_moc|Meetings]]

```dataview  
TABLE description As Description 
From -"3_Resources/Obsidian/Templates" 
WHERE  date >= striptime(this.date)
WHERE  date <= striptime(this.date) + dur(6 days)
WHERE contains(type,"meeting")
Sort file.name ASC
Limit 7  
```
---
## [[0_daily/tasks|Tasks]]
```tasks
status.type is not CANCELLED

filter by function  task.done.moment?.isSame(moment("<% tp.date.weekday("YYYY-MM-DD", 0) %>"), 'week') || (task.scheduled.moment?.isSame(moment("<% tp.date.weekday("YYYY-MM-DD", 0) %>"), 'week') && !task.done.moment)|| (task.scheduled.moment?.isBefore(moment("<% tp.date.weekday("YYYY-MM-DD", 0) %>"), 'week') && !task.done.moment)|| false


sort by description
group by scheduled



```


<%*tp.file.rename(tp.date.weekday("YYYY-MM-DD", 0)+"_to_"+tp.date.weekday("YYYY-MM-DD", 6));%>
<% tp.user.pin_me() %>