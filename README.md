# integrated-simulation
### توضیحات کد به زبان فارسی

این

---

### 1. **تعریف اندیکاتور و پارامترهای اولیه**
```pinescript
//@version=6
indicator('Integrated Simulation with Table', overlay = true, max_bars_back = 499)
```
- **نسخه Pine Script**: نسخه 6 استفاده شده است.
- **نام اندیکاتور**: `'Integrated Simulation with Table'`.
- **overlay = true**: اندیکاتور روی نمودار قیمت نمایش داده می‌شود.
- **max_bars_back = 499**: حداکثر تعداد کندل‌های قابل بررسی برای محاسبات.

---

### 2. **تعریف آرایه‌ها برای ذخیره مقادیر محدوده‌ها**
```pinescript
var array<float> t_upper = array.new_float(10)  // 1 to 10 for upper internal range
var array<float> t_lower = array.new_float(100) // 1 to 100 for lower internal range
var array<float> t_external = array.new_float(500) // For external ranges 100 to 240 and 0 to -260
```
- **آرایه‌ها**: سه آرایه برای ذخیره مقادیر محدوده‌های مختلف تعریف شده‌اند:
  - `t_upper`: برای محدوده داخلی 1 تا 10.
  - `t_lower`: برای محدوده داخلی 1 تا 100.
  - `t_external`: برای محدوده‌های خارجی 100 تا 240 و 0 تا 260-.

---

### 3. **تابع مقداردهی اولیه آرایه‌ها**
```pinescript
f_init_arrays() =>
    // Upper internal range (1 to 10)
    for i = 0 to 9
        array.set(t_upper, i, 1.0 + i * (9.0/9))
    
    // Lower internal range (1 to 100)
    for i = 0 to 99
        array.set(t_lower, i, 1.0 + i * (99.0/99))
    
    // External range values
    for i = 0 to 499
        if i < 140  // For 100 to 240
            array.set(t_external, i, 100.0 + i)
        else        // For 0 to -260
            array.set(t_external, i, 0.0 - (i - 140))
```
- این تابع مقادیر آرایه‌ها را با استفاده از حلقه‌های `for` مقداردهی می‌کند.
- برای `t_upper`، مقادیر از 1 تا 10 به صورت خطی افزایش می‌یابند.
- برای `t_lower`، مقادیر از 1 تا 100 به صورت خطی افزایش می‌یابند.
- برای `t_external`، مقادیر از 100 تا 240 و سپس از 0 تا 260- تنظیم می‌شوند.

---

### 4. **فراخوانی تابع مقداردهی اولیه**
```pinescript
if bar_index == 0
    f_init_arrays()
```
- تابع `f_init_arrays()` تنها در اولین کندل (`bar_index == 0`) فراخوانی می‌شود.

---

### 5. **ایجاد جدول برای نمایش نتایج**
```pinescript
var table resultsTable = table.new(position.bottom_right, 50, 50, border_color = color.white, bgcolor = color.new(color.black, 70))
```
- یک جدول با موقعیت در گوشه پایین‌راست نمودار ایجاد می‌شود.
- جدول دارای 50 سطر و 50 ستون است و رنگ پس‌زمینه آن مشکی با شفافیت 70% است.

---

### 6. **افزودن هدرهای جدول**
```pinescript
if bar_index == 0
    table.cell(resultsTable, 0, 0, 'Range Type', text_color = color.white, bgcolor = color.new(color.blue, 70))
    table.cell(resultsTable, 0, 1, 'Value', text_color = color.white, bgcolor = color.new(color.blue, 70))
    table.cell(resultsTable, 0, 2, 'Level', text_color = color.white, bgcolor = color.new(color.blue, 70))
```
- در اولین کندل، هدرهای جدول شامل "Range Type"، "Value" و "Level" اضافه می‌شوند.

---

### 7. **تابع محاسبه موقعیت**
```pinescript
Calculate() =>
    position_x = 1.0 + 0.5 * bar_index * 0.1
    [position_x]
```
- این تابع یک مقدار `position_x` را بر اساس `bar_index` محاسبه می‌کند.

---

### 8. **محاسبه نقاط سوئینگ و محدوده‌ها**
```pinescript
numCandles = math.floor(position_x)
hh = ta.highest(numCandles)
ll = ta.lowest(numCandles)
```
- `numCandles`: تعداد کندل‌های مورد بررسی برای محاسبه نقاط سوئینگ.
- `hh`: بالاترین قیمت در `numCandles` کندل اخیر.
- `ll`: پایین‌ترین قیمت در `numCandles` کندل اخیر.

---

### 9. **تعریف و محاسبه نقاط سوئینگ**
```pinescript
if high > hh[1] and high > high[3]
    lastSwingHigh := swingHigh
    swingHigh := high
    line.delete(la)
    la := line.new(x1 = bar_index, y1 = swingHigh, x2 = bar_index + numCandles, y2 = swingHigh, color = color.green, width = 2, style = line.style_dotted)

if low < ll[1] and low < low[3]
    lastSwingLow := swingLow
    swingLow := low
    line.delete(lb)
    lb := line.new(x1 = bar_index, y1 = swingLow, x2 = bar_index + numCandles, y2 = swingLow, color = color.red, width = 2, style = line.style_dotted)
```
- اگر قیمت بالا (High) از `hh[1]` و `high[3]` بیشتر باشد، یک نقطه سوئینگ بالا (Swing High) شناسایی می‌شود و یک خط سبز رسم می‌شود.
- اگر قیمت پایین (Low) از `ll[1]` و `low[3]` کمتر باشد، یک نقطه سوئینگ پایین (Swing Low) شناسایی می‌شود و یک خط قرمز رسم می‌شود.

---

### 10. **محاسبه محدوده (Range)**
```pinescript
RANGE_CALC = math.max(swingHigh, swingLow) - math.min(swingHigh, swingLow)
```
- محدوده (`RANGE_CALC`) به عنوان تفاوت بین بالاترین و پایین‌ترین نقطه سوئینگ محاسبه می‌شود.

---

### 11. **رسم خطوط برای سطوح 0.23 و 0.61**
```pinescript
LEVEL_23 = (0.23 * RANGE_CALC) + swingLow
LEVEL_61 = (0.61 * RANGE_CALC) + swingLow

if barstate.islast
    line.delete(LX1)
    LX1 := line.new
