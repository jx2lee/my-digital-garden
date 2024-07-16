---
{"dg-publish":true,"permalink":"/etc/__/refactoring/replace-control-flag-with-break/","noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


TL;DR: if Loop 를 제어하는 변수(flag)가 존재하면, then break 를 이용한다.

### before
![](https://i.imgur.com/KwJB7MK.png)

```javascript
for (const p of people) {
  if (! found) {  // found: flag var
    if ( p === "Don") {
      sendAlert();
      found = true;
    }
```

### after
```javascript
for (const p of people) {
  if ( p === "Don") {
    sendAlert();
    break; // use break!
  }
```
