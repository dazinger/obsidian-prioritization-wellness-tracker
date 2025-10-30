# Planner
```meta-bind-js-view
{global_metadata/tasks#tasks} as options1
{global_metadata/tasks#prioritized_tasks} as options2
{global_metadata/chores#chores} as options3
{global_metadata/daily#d_ref_qs_clim} as options4
{global_metadata/daily#d_ref_qs} as options5
{global_metadata/weekly#w_qs_clim} as options6
{global_metadata/weekly#w_qs} as options7
{global_metadata/weekly#w_prios_clim} as options8
{global_metadata/weekly#w_prios} as options9
---
function processRange(startHour, endHour) {
  const currentHour = new Date().getHours();
  if (startHour < endHour) {
    return currentHour >= startHour && currentHour < endHour;
  }
  return currentHour >= startHour || currentHour < endHour;
}
function buildTable(tRange, tRange_lng, tRange_shrt, numChores,
                    numHPTasks = numChores, maxItems = 10, rangeType = 'givenRange') {
  const currentHour = new Date().getHours();
  const op1 = context.bound.options1.map(x => `option('${x}')`).join(", ");
  const op2_arr = context.bound.options2;
  const op2 = op2_arr.map(x => `option('${x})')`).join(", ");
  const op3 = context.bound.options3.map(x => `option('${x}')`).join(", ");
  const op4 = context.bound.options4;
  const op5 = context.bound.options5.map(x => `title(${x} \n\\[${op4} char max\\])`);
  const op5len = op5.length; // should be divisible by two, else an empty cell appears
  const op6 = context.bound.options6;
  const op7 = context.bound.options7.map(x => `title(${x} \n\\[${op6} char max\\])`);
  const op7len = op7.length; // should equal two
  for (i = 0; i < op7.length; i++) {
    op7[i] = op7[i].replace("'", "");
  }
  const op8 = context.bound.options8;
  const op9 = context.bound.options9.map(x => `option('${x}')`).join(", ");
  var task = '';
  var chore = '';
  var reflection = '';
  var footer = '';
  var defTaskOpt = '';
  var num_dRefs = 0;
  var result = `| Prioritize the week, then tasks, then ${tRange_shrt[1]} chores |`;
  result += ` If you have time, also fill in the text boxes! |`;
  result += `\n| -------------- | --------------------- |`;
  result += `\n| \`VIEW[{memory^tasks_list_button}][text(renderMarkdown)]\` |`;
  result += ` \`VIEW[{memory^${tRange_lng[0]}_chores_list_button}]`;
  result += `[text(renderMarkdown)]\` |`;
  result += `\n| \`VIEW[{memory^w_q1}][text(renderMarkdown)]\` |`;
  result += ` \`VIEW[{memory^w_q2}][text(renderMarkdown)]\` |`;
  footer += `\n\`\`\`meta-bind`;
  footer += `\nVIEW[~~~meta-bind`;
  footer += `\nINPUT\\[textArea(${op7[0]}, limit(${op6})):`;
  footer += `global_metadata/weekly#w_q1_ans\\]`;
  footer += `\n~~~][text(hidden):memory^w_q1]`;
  footer += `\n\`\`\``;
  for (i = 0; i < maxItems; i++) {
    defTaskOpt = context.bound.options2.map(x => x == `${x}`);
    if (i < numChores) {
      task += `\n| Daily Task #${i+1} (HIGH Priority AND URGENT!)<br>`;
      task += `\`INPUT[inlineSelect(${op1}, defaultValue('${op2_arr[i]}'))`;
      task += `:global_metadata/tasks#prioritized_tasks[${i}]]\` |`;
      chore += ` ${tRange_lng[1]} Chore #${i}<br>`;
      chore += `\`INPUT[inlineSelect(${op3}):global_metadata/chores#`;
      chore += `${tRange_lng[1]}_chores[${i}]]\` |`;
      result += task + chore;
      task = '';
      chore = '';
    } else {
      if (i == numChores) {
        result += `\n| \`VIEW[{memory^prioritized_tasks}][text(renderMarkdown)]\` |`;
        result += ` \`VIEW[{memory^chores}][text(renderMarkdown)]\` |`;
        for (j = 0; j < 2; j++) {
          task += `\n| `;
          if (j == 0) {
            var prios = 'Tasks that are HIGH Priority, LOW Urgency<br>Plan these!';
          } else {
            prios = 'Tasks that are LOW Priority, HIGH Urgency<br>Delegate these!';
          }
          task += `${prios}<br><br>`;
          for (k = 0; k < 3; k++) {
            task += `Daily Task #${i+j*3+k+1}<br>`;
            task += `\`INPUT[inlineSelect(${op1}, defaultValue`;
            task += `('${op2_arr[i+j*3+k]}')):global_metadata/tasks`;
            task += `#prioritized_tasks[${i+j*3+k}]]\`<br>`;
          }
          task += ` |`;
          num_dRefs += 1;
          reflection += ` \`VIEW[{memory^d_ref_Q${j}}]`;
          reflection += `[text(renderMarkdown)]\` |`;
          result += task + reflection;
          reflection = '';
          task = '';
        }
      }
      if (i >= numChores && i < maxItems-2) {
        reflection += ` \`VIEW[{memory^d_ref_Q${i-numChores+2}}]`;
        reflection += `[text(renderMarkdown)]\` |`;
      } else if (i == maxItems-2) {
        result += `\n| \`VIEW[{memory^pos_habits}][text(renderMarkdown)]\` |`;
        result += ` \`VIEW[{memory^neg_habits}][text(renderMarkdown)]\` |`;
      }
      result += task + reflection;
      task = '';
      reflection = '';
      footer += `\n\`\`\`meta-bind`;
      footer += `\nVIEW[~~~meta-bind`;
      footer += `\nINPUT\\[textArea(${op5[i-numChores]}, limit(${op4})):`;
      footer += `global_metadata/daily#Q${i-numChores}_Ans\\]`;
      footer += `\n~~~][text(hidden):memory^d_ref_Q${i-numChores}]`;
      footer += `\n\`\`\``;
    }
  }
  footer += `\n# What's on the calendar?`;
  footer += `\n\`\`\`gEvent`;
  footer += `\ntype: week`;
  footer += `\nnavigation: true`;
  if (rangeType == 'givenRange') {
    footer += `\nhourRange: [${tRange[0]}, ${tRange[1]}]`;
  } else if (rangeType == 'fromNow') {
    footer += `\nhourRange: [${currentHour}, ${currentHour + tRange[1] - tRange[0]}]`;
  } else {
    console.log(`We won't ever reach here unless we remove the default value`);
    // maybe add here some buffer time, maybe 1-2 hours before EOR??
  }
  footer += `\ntimespan: 3`;
  footer += `\n\`\`\``;
  result += footer;
  console.log(`Action performed at: ${new Date().toLocaleTimeString()}`);
  return result;
}
function timeRangeTable(ranges, tRanges_lng, tRanges_shrt, numRanges, numChores,
                        flex = 0) {
  var table = "You should be sleeping!";
  var lastRange = true;
  var currentRange = null;
  var tRange_lng = [null, null];
  var tRange_shrt = [null, null];
  for (i = 0; i < (numRanges - 1); i++) {
    ranges[i] = [ranges[i][0], ranges[i][1] + flex];
    if (processRange(ranges[i][0], ranges[i][1])) { // only 1 range current at any time
      tRange_lng = [tRanges_lng[i], tRanges_lng[i+1]];
      tRange_shrt = [tRanges_shrt[i], tRanges_shrt[i+1]];
      lastRange = false;
      currentRange = ranges[i];
    }
  }
  if (lastRange) {
    if (processRange(ranges[i][0], ranges[i][1])) {
      tRange_lng = [tRanges_lng[i], tRanges_lng[0]];
      tRange_shrt = [tRanges_shrt[i], tRanges_shrt[0]];
      lastRange = true;
      currentRange = ranges[i];
    } else { // beyond last range, but before first range
      tRange_lng = [tRanges_lng[0], tRanges_lng[1]];
      tRange_shrt = [tRanges_shrt[0], tRanges_shrt[1]];
      lastRange = false;
      currentRange = [0, ranges[0][0]]; // todo: if ranges[i][1] < 24, what do we do?
      console.log(`range is ${currentRange}`)
    }
  }
  table = buildTable(currentRange, tRange_lng, tRange_shrt, numChores);
  return table;
}
var numRanges = 3;
var tRanges_lng = ["Wakeup", "Lunchtime", "Bedtime"];
var tRanges_shrt = ["WT", "LT", "BT"];
var ranges = [[6, 12], [12, 18], [18, 24]];
//var gCalHourRange = 6;
const str = timeRangeTable(ranges, tRanges_lng, tRanges_shrt, numRanges, 4, 0);
return engine.markdown.create(str);
```
```meta-bind
VIEW[~~~meta-bind
  INPUT\[list(placeholder('Enter priorities for this week!'),
  limit(35)):global_metadata/weekly#w_prios\]
~~~][text(hidden):memory^w_q2]
```
```meta-bind
VIEW[~~~meta-bind
  INPUT\[list(placeholder('Move tasks up/down to prioritize')):
  global_metadata/tasks#prioritized_tasks\]
~~~][text(hidden):memory^prioritized_tasks]
```
```meta-bind
VIEW[~~~meta-bind
  INPUT\[list(placeholder('Add/remove chore')):global_metadata/chores#chores\]
~~~][text(hidden):memory^chores]
```
```meta-bind
VIEW[~~~meta-bind-button
  style: primary
  label: I COMPLETED ALL HIGH-PRIORITY TASKS TODAY
  id: wakeup
  actions:
  - type: updateMetadata
    bindTarget: global_metadata/tasks#high_priority_tasks_complete
    evaluate: false
    value: Yes
~~~][text(hidden):memory^tasks_list_button]
```
```meta-bind
VIEW[~~~meta-bind-button
  style: primary
  label: I COMPLETED ALL WAKEUP CHORES FOR TODAY
  id: wakeup
  actions:
  - type: updateMetadata
    bindTarget: global_metadata/chores#Wakeup_complete
    evaluate: false
    value: Yes
~~~][text(hidden):memory^Wakeup_chores_list_button]
```
```meta-bind
VIEW[~~~meta-bind-button
  style: primary
  label: I COMPLETED ALL LUNCH CHORES FOR TODAY
  id: lunch
  actions:
  - type: updateMetadata
    bindTarget: global_metadata/chores#Lunchtime_complete
    evaluate: false
    value: Yes
~~~][text(hidden):memory^Lunchtime_chores_list_button]
```
```meta-bind
VIEW[~~~meta-bind-button
  style: primary
  label: I COMPLETED ALL CHORES BEFORE BEDTIME
  id: wakeup
  actions:
  - type: updateMetadata
    bindTarget: global_metadata/chores#Bedtime_complete
    evaluate: false
    value: Yes
~~~][text(hidden):memory^Bedtime_chores_list_button]
```
```meta-bind
VIEW[~~~meta-bind
INPUT\[list(title('Positive habits to encourage'), placeholder('Provide a frequency!'), limit(35), multiLine(false)):global_metadata/habits#pos_habits_list\]
~~~][text(hidden):memory^pos_habits]
```
```meta-bind
VIEW[~~~meta-bind
INPUT\[list(title('Poor habits to stop'), placeholder('Habits of thought count, too!'), limit(35), multiLine(false)):global_metadata/habits#neg_habits_list\]
~~~][text(hidden):memory^neg_habits]
```
