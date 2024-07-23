---
date: <% tp.date.weekday("YYYY-MM-DD", 0) %>
type: weekly
tags: "#moc #daily/<% tp.date.now("YYYY") %>/<% tp.date.now("MM") %> #fitness/training "
description: Training log from <% tp.date.weekday("YYYY-MM-DD", 0)%> to <% tp.date.weekday("YYYY-MM-DD", 6)%>
---

[[0_daily/training/<% tp.date.weekday("YYYY-MM-DD", -7) %>_to_<% tp.date.weekday("YYYY-MM-DD", -1) %>|last week]]
[[0_daily/training/<% tp.date.weekday("YYYY-MM-DD", 7) %>_to_<% tp.date.weekday("YYYY-MM-DD", 13) %>|next week]]

## Training log

```dataviewjs
const tracked = {};
const tagsRegex = /#\S+(?:\s\S+)*/g;

// Define the start and end dates for the desired week
const weekStartDate = dv.current().file.frontmatter.date;
const weekEndDate = moment(weekStartDate).add(6, 'days').format('YYYY-MM-DD');

// Gather data
dv.pages('"0_daily"').file.lists
  .where(x => x.section.subpath === "Training log").array() 
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
        
        let kilometers = null; // Initialize kilometers as null
        
        // Check if the entry has the tag for running
        if (tags && tags.includes("#fitness/training/running")) {

// Extract kilometers from the description
const kmRegex = /(?:\[\[.*?(\d+(\.\d+)?) km.*?\]\]|(\d+(\.\d+)?) km(?:\s*\[\[.*?\]\])?)/; // Regex to match kilometers with or without links
const kmMatch = description.match(kmRegex);
if (kmMatch) {
  // Parse kilometers as float from either capture group
  kilometers = parseFloat(kmMatch[1] || kmMatch[3]);
}
        }

        if (!tracked[week]) tracked[week] = {};
        if (tracked[week][date]) {
          tracked[week][date].entries.push({
            path: x.path,
            start: start.format('HH:mm'),
            end: end.format('HH:mm'),
            minutes: minutes,
            tags: tags || [],
            description: description || '', // Initialize description as an empty string
            kilometers: kilometers // Assign kilometers to entry
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
              description: description || '', // Initialize description as an empty string
              kilometers: kilometers // Assign kilometers to entry
            }],
            minutes: minutes,
            tags: tags || [],
            description: description || '' // Initialize description as an empty string
          };
        }
      }
    }
  });

const table = [];
let totalWeekMinutes = 0;
let totalWeekKilometers = 0; // Initialize total kilometers for the week

// Calculate total minutes and kilometers before iterating through daily entries
Object.keys(tracked).sort((a, b) => b.localeCompare(a))
  .forEach(weekDate => {
    const weekEntries = tracked[weekDate];

    Object.keys(weekEntries).forEach(date => {
      totalWeekMinutes += weekEntries[date].minutes;

      // Accumulate kilometers for the week
      weekEntries[date].entries.forEach(entry => {
        if (entry.kilometers !== null) {
          totalWeekKilometers += entry.kilometers;
        }
      });
    });
  });

// Push total minutes and kilometers row to the top of the table
table.push([
  '**Total**', 
  '**' + totalWeekMinutes + '**', 
  '**' + totalWeekKilometers.toFixed(2) + '**', // Display total kilometers with 2 decimal places
  ''
]);
// Horizontal line after the total minutes row
// table.push(['----', '----', '----', '']);

// Iterate through tracked entries to fill the table
Object.keys(tracked).sort((a, b) => b.localeCompare(a))
  .forEach(weekDate => {
    const weekEntries = tracked[weekDate];

    // Sort daily values within the week summary in descending order by date
    const sortedDates = Object.keys(weekEntries).sort((a, b) => b.localeCompare(a));

    sortedDates.forEach(date => {
      const dailyEntry = weekEntries[date];
      const link = `[[${dailyEntry.path}#Training log|${moment(date).format('dddd D MMMM')}]]`;

      // Push daily summary above the entries for each day
      table.push(['**' + link + '**', '**' + dailyEntry.minutes + '**', '']);

      // Push individual time entries (sorted in descending order)
      dailyEntry.entries.sort((a, b) => moment(b.start, 'HH:mm').diff(moment(a.start, 'HH:mm'))).forEach(entry => {
        table.push([
          `${entry.start}-${entry.end}`,
          entry.minutes,
          entry.kilometers !== null ? entry.kilometers.toFixed(2) : '', // Remove 'km' and only report the digits
          entry.description
        ]);
      });
    });
  });

dv.table(['Date', 'Minutes', 'km', 'Description'], table);


```



---
## Training plan
[time calculator](https://www.calculator.net/time-calculator.html?tcday1=&tchour1=06&tcminute1=52&tcsecond1=&Op=%2B&tcday2=&tchour2=1&tcminute2=27&tcsecond2=&tcday3=&tchour3=&tcminute3=&tcsecond3=&ctype=1&x=Calculate)
### Day 1




### Day 2




### Day 3



### Day 4


### Day 5


### Day 6


### Day 7





<%*tp.file.rename(tp.date.weekday("YYYY-MM-DD", 0)+"_to_"+tp.date.weekday("YYYY-MM-DD", 6));%>
<% tp.user.pin_me() %>