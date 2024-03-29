﻿---
title: Fall 2019 Schedule
...

The following is the current best guess at course pacing and topics.
It may be adjusted as the semester progresses to reflect actual pacing and uptake of material.

<hr/>

<style id="schedule-css">

#schedule td, #schedule th { padding: 0ex; }

table.calendar { 
    border-collapse: collapse; 
    width: 100%; 
    background: rgba(0,0,0,0.125); 
    border: 0.5ex solid rgba(0,0,0,0);
    border-radius: 1.5ex; 
}
table.calendar td:empty { padding: 0; height: 4em; }
table.calendar td { border: 0.25ex solid rgba(0,0,0,0); }
table.calendar span.date { 
    font-size: 70.7%;
    padding-left: 0.5ex;
    float:right;
    margin-top:-0.5ex;
}
table.calendar div.wrapper { 
    background: white;
    border-radius: 1ex;
    padding: .5ex;
    box-sizing:border-box; 
    width: 100%;
    height: 100%;
    min-height:5em; 
    overflow: hidden;
}
table.calendar div.wrapper div {
    padding: 0 0.5ex 0 0.5ex;
    margin: 0 -0.5ex 0 -0.5ex;
}
table.calendar div.wrapper div:first-child {
    padding-top: 0.5ex;
    margin-top: -0.5ex;
}
table.calendar div.wrapper div:last-child {
    padding-bottom: 0.5ex;
    margin-bottom: -0.5ex;
}


table.agenda, table.agenda tbody { display: block; }
table.agenda tr {
    display: block; border-top: thick solid grey;
    min-height: 2em;
}
table.agenda td {
    display: table; border-top: thin solid grey; width: 100%;
    padding: 0;
}
table.agenda td:empty { display: none; }
table.agenda span.date.w0:before { content: "Sun "; }
table.agenda span.date.w1:before { content: "Mon "; }
table.agenda span.date.w2:before { content: "Tue "; }
table.agenda span.date.w3:before { content: "Wed "; }
table.agenda span.date.w4:before { content: "Thu "; }
table.agenda span.date.w5:before { content: "Fri "; }
table.agenda span.date.w6:before { content: "Sat "; }
table.agenda span.date {
    font-size: 70.7%; width:7em;
    vertical-align: middle; 
    display: table-cell;
    padding: 0 0.5ex;
}
table.agenda div.wrapper { display: table-row; }
table.agenda div.events { display: table-cell; vertical-align: middle; }

.assignment:before { content: "due: "; font-size: 70.7%; }
.lab:before { content: "lab: "; font-size: 70.7%; }
.lab summary:before { content: "lab: "; font-size: 70.7%; }
details.lab:before { content: ""; }
small { opacity: 0.5; }
.special, .exam, .assignment:before { background: rgba(255,127,0,0.25); opacity: 0.75; }
span.date { font-family:monospace; }
details { padding-left: 1em; }
summary { margin-left: -1em; }

.day.past { opacity: 0.707; }
.day.today .wrapper { box-shadow: 0 0 0.5ex 0.5ex grey; }
.agenda .day.today .wrapper { margin: 0.5ex 0;}

</style>


<p>View as 
<label><input type="radio" name="viewmode" onchange="viewmode(this)" value="calendar" id="viewmode=calendar"> calendar</label>
or
<label><input type="radio" name="viewmode" onchange="viewmode(this)" checked value="agenda" id="viewmode=agenda"> agenda</label>;
<label><input type="checkbox" name="showpast" onclick="showPast(this)" checked id="showpast"> show past</label>;
readings can be <input type="button" value="shown" onclick="document.querySelectorAll('details').forEach(x => x.setAttribute('open','open'))"></input> or <input type="button" value="hidden" onclick="document.querySelectorAll('details').forEach(x => x.removeAttribute('open'))"></input> as a whole, or clicked on individually to toggle visibility.
</p>



{#include schedule-old.html}


<script src="schedule.js"></script>


To subscribe to the above calendar, add <http://www.cs.virginia.edu/luther/COA1/F2019/cal.ics> to your calender application of choice.

<hr/>

The <a href="http://www.virginia.edu/registrar/exams.html#1198">final exam schedule</a> puts our final Saturday 14 December at 9:00 am. The final is an in-person on-paper exam administered in the usual classroom.

