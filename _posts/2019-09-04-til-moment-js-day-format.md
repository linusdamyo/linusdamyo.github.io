---
layout: post
title:  "TIL : moment.js 요일 format"
categories: javascript
---

항상 ddd로 썼는데..

```javascript
  > const moment = require('moment')
  > moment().format('dd')
  'Th'
  > moment().format('ddd')
  'Thu'
  > moment().locale('ko').format('d')
  '4'
  > moment().locale('ko').format('dd')
  '목'
  > moment().locale('ko').format('ddd')
  '목'
  > moment().locale('ko').format('dddd')
  '목요일'
```
