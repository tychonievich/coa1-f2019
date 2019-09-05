---
title: Office Hours
...

Professor offices hours are in the professor's office.
If I'm engaged with a student, don't just wait silently in the hall; ask me if you can come in and join the other student(s) there (often the answer will be yes, and even if no I'll know you are waiting).

TA office hours are in Thornton Stacks (A-235), and are requested by using the [office hour queue](https://kytos.cs.virginia.edu/ohq/?c=coa1).
The queue is open even when TAs are not present; see [contact information](policies.html#contact) for when to expect TAs in office hours.

<div style="display:table; font-size:200%; margin: 1em auto; padding:1ex; box-shadow: 0 1px 10px rgba(0,0,0,.1); border: thin solid #eee; border-radius:1ex; background-image: linear-gradient(to bottom, #ffffff, #f2f2f2);">[Get TA Help](https://kytos.cs.virginia.edu/ohq/?c=coa1)</div>

<div id="cal-oh"></div>

<script src="cal-oh.js" type="text/javascript"></script>
<script type="text/javascript">//<!--
now = new Date().toISOString()
week = new Date(); week.setDate(week.getDate() + 7); week = week.toISOString()
within = document.getElementById('cal=oh')
oh_feed.forEach(x => {
    if (x.end < now || x.start > week) return;
    console.log(x)
    let entry = document.createElement('div')
    entry.classList.add('oh')
    entry.classList.add(x['title'].split(' ')[0] == 'TA' ? 'ta' : 'faculty')
    if ('link' in x) {
        let a = document.createElement('a')
        a.href = x['link']
        a.innerText = x['title']
        entry.appendChild(a);
    } else {
        entry.innerText = x['title']
    }
    within.appendChild(entry)
})
//--></script>
