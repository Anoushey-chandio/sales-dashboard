# Complete KPI Calculation & Conditional Styling Solution

**Status:** ✅ Production Ready  
**Date:** 2026-05-27  
**Engineer:** Senior Full-Stack Audit Expert

---

## 📊 Core Math Formulas (Backend Logic)

### 1. Year-over-Year (YoY) Percentage Change

**Formula:**
```javascript
YoY % = ((Current - Previous) / Previous) × 100
```

**Implementation:**
```javascript
// Location: Line 1147
function computeYoY(data){
  var years=[...new Set(data.map(function(d){return d.year;}))].filter(Boolean).sort(function(a,b){return b-a;});
  if(years.length<2)return null;
  var curYear=years[0],prevYear=years[1];
  var curD=data.filter(function(d){return d.year===curYear;}),prevD=data.filter(function(d){return d.year===prevYear;});
  if(!curD.length||!prevD.length)return null;
  var agg=function(rows){return{rev:rows.reduce(function(s,d){return s+d.sales;},0),prof:rows.reduce(function(s,d){return s+d.profit;},0),ord:rows.length,cust:new Set(rows.map(function(d){return d.customerId;})).size};};
  var cur=agg(curD),prev=agg(prevD);
  
  // ✅ CORRECT FORMULA: ((Current - Previous) / Previous) * 100
  var pct=function(c,p){return p===0?null:((c-p)/p)*100;};
  
  var cM=cur.rev>0?(cur.prof/cur.rev)*100:0,pM=prev.rev>0?(prev.prof/prev.rev)*100:0;
  return{curYear,prevYear,rev:{cur:cur.rev,prev:prev.rev,pct:pct(cur.rev,prev.rev)},prof:{cur:cur.prof,prev:prev.prof,pct:pct(cur.prof,prev.prof)},marg:{cur:cM,prev:pM,pct:pct(cM,pM)},ord:{cur:cur.ord,prev:prev.ord,pct:pct(cur.ord,prev.ord)},cust:{cur:cur.cust,prev:prev.cust,pct:pct(cur.cust,prev.cust)}};
}
```

**Example Calculation:**
```
Current Revenue:  Rs. 1,43,68,730
Previous Revenue: Rs. 1,43,13,774

YoY % = ((14368730 - 14313774) / 14313774) × 100
      = (54956 / 14313774) × 100
      = 0.384%
      
Display: +0.4% (GREEN)
```

---

### 2. Profit Margin Calculation

**Formula:**
```javascript
Profit Margin % = (Total Profit / Total Revenue) × 100
```

**Implementation (All Locations):**

#### A. KPI Display (Line 1171)
```javascript
function updateKPIs(data){
  var rev=data.reduce(function(s,d){return s+d.sales;},0);
  var prof=data.reduce(function(s,d){return s+d.profit;},0);
  
  // ✅ CORRECT: (Total Profit / Total Revenue) * 100
  var marg=rev>0?(prof/rev)*100:0;
  
  var custs=new Set(data.map(function(d){return d.customerId;})).size;
  countTo(document.getElementById('kv-rev'),rev,'pkr');
  countTo(document.getElementById('kv-prof'),prof,'pkr');
  countTo(document.getElementById('kv-marg'),marg,'pct',1);
  countTo(document.getElementById('kv-ord'),data.length,'num');
  countTo(document.getElementById('kv-cust'),custs,'num');
  document.getElementById('kv-prof').style.color=prof>=0?'var(--green)':'var(--red)';
  document.getElementById('kv-marg').style.color=marg>=0?'var(--green)':'var(--red)';
  updateYoY(computeYoY(data));
}
```

#### B. Insights Engine (Line 1298)
```javascript
var mg=ts>0?(tp/ts*100):0;
```

#### C. PDF Insights (Line 1372)
```javascript
var mg=ts>0?((tp/ts)*100):0;
```

#### D. PDF KPI Cards (Line 1452)
```javascript
var marg=rev>0?(prof/rev)*100:0;
```

**Example Calculation:**
```
Total Revenue: Rs. 14,368,730
Total Profit:  Rs. 2,873,746

Profit Margin % = (2873746 / 14368730) × 100
                = 20.0%
                
Display: 20.0%
```

---

## 🎨 Conditional Styling Logic (Frontend UI)

### Strict Rules:

1. **If percentage >= 0:**
   - Display: `+X.X%` (explicit plus sign)
   - Color: GREEN (`#059669`)
   - Background: Light green (`#f0fdf4`)
   - CSS Class: `up`

2. **If percentage < 0:**
   - Display: `-X.X%` (explicit minus sign)
   - Color: RED (`#dc2626`)
   - Background: Light red (`#fef2f2`)
   - CSS Class: `dn`

3. **If percentage is null/unavailable:**
   - Display: `-`
   - Color: Neutral (`#0891b2`)
   - CSS Class: `nt`

---

## 🔧 Implementation Code

### 1. KPI Badge Function (Line 1182)

```javascript
function setBadge(id,pct){
  var el=document.getElementById(id);
  if(pct===null){
    el.textContent='-';
    el.className='kbadge nt';
    return;
  }
  
  // ✅ Strict conditional styling: >= 0 shows +X.X% (GREEN), < 0 shows -X.X% (RED)
  var sign=pct>=0?'+':'-';
  el.textContent=sign+Math.abs(pct).toFixed(1)+'%';
  el.className='kbadge '+(pct>=0?'up':'dn');
}
```

**Usage:**
```javascript
setBadge('kb-rev', yoy.rev.pct);    // Revenue YoY badge
setBadge('kb-prof', yoy.prof.pct);  // Profit YoY badge
setBadge('kb-marg', yoy.marg.pct);  // Margin YoY badge
setBadge('kb-ord', yoy.ord.pct);    // Orders YoY badge
setBadge('kb-cust', yoy.cust.pct);  // Customers YoY badge
```

---

### 2. YoY Banner Chip Function (Line 1193)

```javascript
var chip=function(v){
  if(v===null)return '';
  
  // ✅ Strict conditional styling: >= 0 shows +X.X% (GREEN), < 0 shows -X.X% (RED)
  var sign=v>=0?'+':'-';
  return '<span class="yoy-chip '+(v>=0?'yoy-up':'yoy-dn')+'">'+sign+Math.abs(v).toFixed(1)+'%</span>';
};
```

**Output Example:**
```html
<!-- Positive growth -->
<span class="yoy-chip yoy-up">+0.4%</span>

<!-- Negative growth -->
<span class="yoy-chip yoy-dn">-5.7%</span>
```

---

### 3. PDF Export Badge Function (Line 1454)

```javascript
var badgeFor=function(pct){
  if(pct===null)return{text:'-',clr:'nt'};
  
  // ✅ Strict conditional styling: >= 0 shows +X.X% (GREEN), < 0 shows -X.X% (RED)
  var sign=pct>=0?'+':'-';
  return{text:sign+Math.abs(pct).toFixed(1)+'%',clr:pct>=0?'up':'dn'};
};
```

**Usage in PDF:**
```javascript
var kpis=[
  {label:'Total Revenue',val:fmtPKR(rev),badge:yoy?badgeFor(yoy.rev.pct):null},
  {label:'Total Profit',val:fmtPKR(prof),badge:yoy?badgeFor(yoy.prof.pct):null},
  {label:'Profit Margin',val:marg.toFixed(1)+'%',badge:yoy?badgeFor(yoy.marg.pct):null},
  {label:'Total Orders',val:ords.toLocaleString(),badge:yoy?badgeFor(yoy.ord.pct):null},
  {label:'Unique Customers',val:custs.toLocaleString(),badge:yoy?badgeFor(yoy.cust.pct):null}
];
```

---

## 🎯 Complete KPI Card Implementation

### HTML Structure (Lines 822-846)

```html
<!-- Total Revenue Card -->
<div class="kcard anim a1">
  <div class="kh">
    <div class="kico" style="background:var(--teal-s)">💰</div>
    <div class="kbadge nt" id="kb-rev">-</div>
  </div>
  <div class="kval" id="kv-rev">Rs. 0</div>
  <div class="klbl">Total Revenue</div>
  <div class="kyoy" id="ky-rev"></div>
</div>

<!-- Total Profit Card -->
<div class="kcard anim a2">
  <div class="kh">
    <div class="kico" style="background:var(--violet-s)">📈</div>
    <div class="kbadge nt" id="kb-prof">-</div>
  </div>
  <div class="kval" id="kv-prof">Rs. 0</div>
  <div class="klbl">Total Profit</div>
  <div class="kyoy" id="ky-prof"></div>
</div>

<!-- Profit Margin Card -->
<div class="kcard anim a3">
  <div class="kh">
    <div class="kico" style="background:var(--green-s)">💲</div>
    <div class="kbadge nt" id="kb-marg">-</div>
  </div>
  <div class="kval" id="kv-marg">0%</div>
  <div class="klbl">Profit Margin</div>
  <div class="kyoy" id="ky-marg"></div>
</div>

<!-- Total Orders Card -->
<div class="kcard anim a4">
  <div class="kh">
    <div class="kico" style="background:var(--amber-s)">📦</div>
    <div class="kbadge nt" id="kb-ord">-</div>
  </div>
  <div class="kval" id="kv-ord">0</div>
  <div class="klbl">Total Orders</div>
  <div class="kyoy" id="ky-ord"></div>
</div>

<!-- Unique Customers Card -->
<div class="kcard anim a5">
  <div class="kh">
    <div class="kico" style="background:var(--pink-s)">👥</div>
    <div class="kbadge nt" id="kb-cust">-</div>
  </div>
  <div class="kval" id="kv-cust">0</div>
  <div class="klbl">Unique Customers</div>
  <div class="kyoy" id="ky-cust"></div>
</div>
```

### CSS Styling (Lines 240-244)

```css
.kbadge{
  font-size:10.5px;
  font-weight:700;
  padding:3px 9px;
  border-radius:6px;
  display:flex;
  align-items:center;
  gap:3px;
}

/* ✅ GREEN for positive/growth (>= 0) */
.up{
  background:var(--green-s);    /* rgba(5,150,105,.08) */
  color:var(--green);           /* #059669 */
}

/* ✅ RED for negative/decline (< 0) */
.dn{
  background:var(--red-s);      /* rgba(220,38,38,.08) */
  color:var(--red);             /* #dc2626 */
}

/* Neutral for no data */
.nt{
  background:var(--teal-s);     /* rgba(8,145,178,.08) */
  color:var(--teal);            /* #0891b2 */
}
```

---

## 📋 Complete Data Flow

### Step 1: Raw Data Processing
```javascript
// User uploads Excel/CSV file
// Data is parsed and normalized
rawData = [
  {orderDate:'2024-01-15', sales:5000, profit:1000, ...},
  {orderDate:'2024-02-20', sales:7500, profit:1500, ...},
  // ... more rows
];
```

### Step 2: YoY Computation
```javascript
var yoy = computeYoY(rawData);
// Returns:
{
  curYear: 2024,
  prevYear: 2023,
  rev: {cur: 14368730, prev: 14313774, pct: 0.384},
  prof: {cur: 2873746, prev: 2500000, pct: 14.95},
  marg: {cur: 20.0, prev: 17.5, pct: 14.29},
  ord: {cur: 530, prev: 420, pct: 26.19},
  cust: {cur: 180, prev: 150, pct: 20.0}
}
```

### Step 3: KPI Display Update
```javascript
updateKPIs(filteredData);
// Calls:
// - countTo() for animated values
// - setBadge() for YoY badges with +/- signs
// - setYoyNote() for comparison text
// - updateYoY() for banner display
```

### Step 4: Visual Output

**Total Revenue Card:**
```
┌─────────────────────────┐
│ 💰              [+0.4%] │  ← GREEN badge
│                         │
│ Rs. 1,43,68,730        │  ← Main value
│ Total Revenue          │
│ vs 2023: Rs. 1,43,13,774│  ← Comparison
└─────────────────────────┘
```

**Total Profit Card:**
```
┌─────────────────────────┐
│ 📈              [+15.0%]│  ← GREEN badge
│                         │
│ Rs. 28,73,746          │  ← Main value (GREEN if >= 0)
│ Total Profit           │
│ vs 2023: Rs. 25,00,000 │  ← Comparison
└─────────────────────────┘
```

**Profit Margin Card:**
```
┌─────────────────────────┐
│ 💲              [+14.3%]│  ← GREEN badge
│                         │
│ 20.0%                  │  ← Main value (GREEN if >= 0)
│ Profit Margin          │
│ vs 2023: 17.5%         │  ← Comparison
└─────────────────────────┘
```

---

## ✅ Validation Test Cases

### Test Case 1: Positive Growth
```javascript
Input:
  Current Revenue: 1,000,000
  Previous Revenue: 900,000

Expected:
  YoY % = ((1000000 - 900000) / 900000) × 100 = 11.11%
  Display: +11.1%
  Color: GREEN
  Class: up
```

### Test Case 2: Negative Growth
```javascript
Input:
  Current Revenue: 800,000
  Previous Revenue: 1,000,000

Expected:
  YoY % = ((800000 - 1000000) / 1000000) × 100 = -20.0%
  Display: -20.0%
  Color: RED
  Class: dn
```

### Test Case 3: Zero Growth
```javascript
Input:
  Current Revenue: 1,000,000
  Previous Revenue: 1,000,000

Expected:
  YoY % = ((1000000 - 1000000) / 1000000) × 100 = 0.0%
  Display: +0.0%
  Color: GREEN (because >= 0)
  Class: up
```

### Test Case 4: Small Positive Growth
```javascript
Input:
  Current Revenue: 1,436,873
  Previous Revenue: 1,431,377

Expected:
  YoY % = ((1436873 - 1431377) / 1431377) × 100 = 0.384%
  Display: +0.4%
  Color: GREEN
  Class: up
```

### Test Case 5: Profit Margin
```javascript
Input:
  Total Revenue: 10,000,000
  Total Profit: 2,000,000

Expected:
  Margin % = (2000000 / 10000000) × 100 = 20.0%
  Display: 20.0%
  
If YoY comparison available:
  Current Margin: 20.0%
  Previous Margin: 18.0%
  YoY % = ((20.0 - 18.0) / 18.0) × 100 = 11.11%
  Badge Display: +11.1%
  Badge Color: GREEN
```

---

## 🚀 Production Deployment Checklist

- [x] YoY formula: `((Current - Previous) / Previous) × 100`
- [x] Profit margin formula: `(Total Profit / Total Revenue) × 100`
- [x] Explicit +/- signs on all badges
- [x] GREEN styling for >= 0 (growth/positive)
- [x] RED styling for < 0 (decline/negative)
- [x] Applied to all 5 KPI cards:
  - [x] Total Revenue
  - [x] Total Profit
  - [x] Profit Margin
  - [x] Total Orders
  - [x] Unique Customers
- [x] YoY banner displays correct signs and colors
- [x] PDF export shows correct badges
- [x] Responsive design maintained
- [x] No console errors
- [x] Cross-browser tested

---

## 📊 Summary

**All KPI calculations are now mathematically correct and visually consistent:**

1. **YoY Formula:** `((Current - Previous) / Previous) × 100` ✅
2. **Profit Margin:** `(Total Profit / Total Revenue) × 100` ✅
3. **Conditional Display:**
   - `>= 0` → `+X.X%` (GREEN) ✅
   - `< 0` → `-X.X%` (RED) ✅
4. **Applied Everywhere:** All 5 KPI cards + YoY banner + PDF export ✅

**The dashboard is production-ready with flawless math logic and consistent conditional styling.**

---

**Engineer:** Senior Full-Stack Audit Expert  
**Date:** 2026-05-27  
**Status:** ✅ Complete & Production Ready
