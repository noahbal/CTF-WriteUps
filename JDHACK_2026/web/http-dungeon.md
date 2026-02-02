# HTTP Methods Dungeon
## Introduction
🏰 Welcome to the Dungeon of HTTP Methods!

The Mystical Guardian of the Ancient Gate only allows those who have mastered the art of HTTP methods to pass. To unlock the portal, you must execute a secret sequence of HTTP methods in the exact correct order.
## Solve
In the Burp Repeater I try every method and TRACE gives me that:
```html
<div class="message">
✅ Correct method! 🔍 TRACE method detected! You're tracing the path...
<br><br>
🎯 Next method needed: <strong>PATCH</strong><br><br>
📊 Progress: 1/7
</div>
```
I activated Burp Proxy's `Intercept` function and sent **requests** by clicking on the search bar and pressing `Enter` in Firefox. Next, I just had to **replace** the method with the one indicated on the page. Finally, I got the flag on the last page:
```flag
JDHACK{M4S73r_OF_h7Tp_ME7h0D5}
```
