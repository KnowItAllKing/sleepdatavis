---
theme: dashboard
title: Sleep data
toc: false
---

# Sleep data

```js
const data = FileAttachment('data/kaisleep.csv').csv({ typed: true });
```

```js
const processedData = data.flatMap((d) => {
  const bedtime = new Date(d.bedtime);
  const waketime = new Date(d.waketime);
  const sleepDuration = (waketime - bedtime) / (1000 * 60 * 60); // Duration in hours

  // If the waketime is earlier than bedtime, it indicates crossing midnight
  let sessions = [];
  if (waketime <= bedtime) {
    waketime.setDate(waketime.getDate() + 1); // Adjust for crossing midnight
  }
  sessions.push({
    date: bedtime,
    endDate: waketime,
    bedTimeHours: bedtime.getHours() + bedtime.getMinutes() / 60, // Start time of day
    wakeTimeHours: (waketime.getHours() + waketime.getMinutes() / 60) % 24, // Normalize to 24-hour, in case of crossing midnight
    sleepDuration,
    wakingBPM: d.wakingBPM,
  });

  return sessions;
});

const startDate = d3.min(processedData, (d) => d.date);
const endDate = d3.max(processedData, (d) => d.date);

const generateMonthlyTicks = (start, end) => {
  let current = new Date(start.getFullYear(), start.getMonth(), 1); // Set to the first day of the start month
  const endMonth = new Date(end.getFullYear(), end.getMonth(), 1);
  const ticks = [];

  while (current <= endMonth) {
    ticks.push(new Date(current)); // Add the first day of the current month
    current.setMonth(current.getMonth() + 1); // Move to the next month
  }

  return ticks;
};

const xTicks = generateMonthlyTicks(startDate, endDate);
```

## Time of Day Spent Sleeping

### with redder being higher waking heart rate, bluer being lower

```js
display(
  Plot.plot({
    x: {
      type: 'utc',
      domain: d3.extent(processedData, (d) => d.date),
      label: 'Date',
      tickValues: xTicks,
      tickFormat: d3.utcFormat('%b %Y'), // Formatting as "Month Year"
    },
    y: {
      domain: [0, 24],
      label: 'Time of Day (Hours)',
    },
    color: {
      type: 'linear', // Define a linear scale for color
      domain: d3.extent(processedData, (d) => d.wakingBPM), // Use the extent of wakingBPM as the domain
      range: ['blue', 'red'], // Range from blue (low) to red (high)
    },
    marks: [
      Plot.rectY(processedData, {
        x: (d) => d.date,
        x2: (d) => d.endDate,
        y: (d) => d.bedTimeHours,
        y2: (d) => d.wakeTimeHours,
        fill: (d) => d.wakingBPM, // Map wakingBPM to the color scale
        title: (d) =>
          `Sleep from ${d3.timeFormat('%H:%M')(
            new Date(d.bedtime)
          )} to ${d3.timeFormat('%H:%M')(new Date(d.waketime))}`,
      }),
    ],
  })
);
```

## Amount of Time Sleeping

```js
display(
  Plot.plot({
    x: {
      type: 'utc',
      domain: d3.extent(processedData, (d) => d.date),
      label: 'Date',
      tickValues: xTicks,
      tickFormat: d3.utcFormat('%b %Y'),
    },
    y: {
      label: 'Hours Slept',
    },
    color: {
      type: 'linear',
      domain: d3.extent(processedData, (d) => d.wakingBPM),
      range: ['blue', 'red'],
    },
    marks: [
      Plot.rectY(processedData, {
        x: (d) => d.date,
        x2: (d) => d.endDate,
        y: 0,
        y2: (d) => d.sleepDuration,
        fill: (d) => d.wakingBPM,
        title: (d) =>
          `Slept ${d.sleepDuration} hours with waking BPM: ${d.wakingBPM}`,
      }),
    ],
  })
);
```
