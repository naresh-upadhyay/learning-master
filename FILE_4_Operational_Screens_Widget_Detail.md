# EduVerse Pro v4.0 — FILE 4: Operational Screens — Widget-Level Detail
## Agent Scope: All Non-Dashboard Screens — Attendance · Fee · Visitor · Exam · Transport · Privilege · AI Log · Analytics · Student Profile · Admissions · Staff · Hostel · Health · Library · Inventory · Communication · Settings
**Build UI only. Use repository classes for data. AiFabButton on EVERY screen (bottom-right, 24dp).**

---

## SCREEN 1: MARK ATTENDANCE SCREEN
**Route:** `/attendance/mark?classId={id}`
**Access:** L3-L5 (teacher owns the session)

### App Bar
- Title: "[Class Name] [Section] — [Subject if specific, else 'General']" 16sp bold white on navyBlue
- Sub-title: "[Date] • Period [X]" 12sp white 70%
- Right actions: [Camera icon] (switch to face mode) [More ⋯]

### Mode Selector (TabBar — 3 tabs)
- Height: 44dp. Full width. Active tab: `royalBlue` underline 3dp
- Tab 1: "✋ Manual" — default
- Tab 2: "📷 Face Scan" — camera-based
- Tab 3: "📊 Bulk" — copy from previous session

### Mode 1: Manual Attendance

#### Search & Filter Bar
- Height: 48dp. Outlined search field 40dp. Placeholder: "Search student name or roll no"
- Right of search: Filter chip row: [All] [Present] [Absent] [Late] [Not Marked]

#### Student List (ListView.builder)
- Each item: 72dp height. Card with 1dp border `borderDefault`. Rounded 12dp.
- Left: Roll number badge (circular, 32dp, `royalBlue` bg, white text 12sp bold)
- Center: [Student name 14sp bold] [Parent: Father name 12sp grey]
- Right: Attendance status selector — 3 icon buttons:
  - ✓ (Present): tap → `teal` fill
  - ✗ (Absent): tap → `red` fill
  - ⏱ (Late): tap → `orange` fill
  - Default (not marked): all buttons grey outlined
- Swipe left: quick-mark absent. Swipe right: quick-mark present.
- Long press: show reason text field (for absent/late)

#### Quick Actions Footer (sticky, above FAB)
- Height: 56dp. `white` bg, top shadow.
- [Mark All Present] (outlined, teal) | [Progress: 28/42 marked] | [Submit →] (filled, navyBlue)
- Submit button disabled until ALL students marked. Shows count of unmarked.

#### Submit Flow
1. [Submit →] tap → validation (all marked? yes proceed)
2. Confirmation dialog: "Submit attendance for [Class]? 38 Present, 4 Absent. Cannot edit after 30 minutes."
3. [Cancel] | [Submit Attendance]
4. Success: green checkmark animation (Lottie), toast "Attendance submitted ✓"
5. Absent students' parents get push notification automatically (via Edge Function)

### Mode 2: Face Scan Attendance
#### Camera Viewfinder
- Full-screen camera feed. White face detection box overlay (animated → green on match)
- Top overlay: "[X] students remaining — [Student name] detected" 
- Face match: display student name + photo thumbnail + [✓ Confirm] [✗ Wrong] buttons
- Below camera: Student row of remaining unscanned (horizontal scroll thumbnails)
- Multiple face detection: highlight closest, cycle through detected

### Mode 3: Bulk Copy
- "Copy from previous class:" — session selector dropdown
- Confirmation: shows diff (students who were absent last time are highlighted)
- [Apply & Review] → goes to manual mode for final verification

---

## SCREEN 2: FEE COLLECTION SCREEN
**Route:** `/fee/collection`
**Access:** L3-L5 with `fee.collect` permission

### Student Search
- Top: Prominent search bar (full width, 52dp, `softSurface` bg, outlined)
- Placeholder: "Search by name, admission no., or phone"
- Results: Name + Class + Outstanding balance chip (red if overdue)
- Recent: "Recent Transactions" mini list

### Student Fee Panel (after selection)

#### Student Info Card (compact — 80dp)
- [Avatar 40dp] [Name 14sp bold] [Class chip] [Admission No. 12sp grey] [Balance: ₹12,400 — large, red if overdue, green if clear]

#### Instalment Timeline
- Card: Title "Fee Instalments — 2025-26"
- Each instalment row (52dp):
  - [Instalment label: Q1/Q2/Annual] [Due date] [Amount: ₹X,XXX] [Status chip] [Paid date if paid]
  - Status chips: "Paid ✓" (tealSoft/teal) | "Due" (orangeSoft/orange) | "Overdue" (redSoft/red) | "Waived" (grey)
  - Tap unpaid instalment: selects it for payment (checkbox behavior)
- Total section: [Selected: ₹X,XXX] [Late fee: +₹XXX] [Total to Pay: ₹X,XXX]

#### Payment Method Selector
- Chip row (horizontal scroll):
  - [💳 UPI] [💵 Cash] [🏦 Cheque] [🔄 NEFT] [📄 DD] [🏛 BBPS] [Razorpay]
  - Selected chip: `navyBlue` bg, white text
- UPI selected: shows sub-options:
  - [Enter Parent VPA] text field + [Send Request] button
  - [Generate QR Code] button — generates UPI QR with amount
- Cash selected: just amount entry (no external action needed)
- Cheque selected: [Cheque No.] [Bank name] [Cheque Date] fields appear

#### Amount Entry
- Large numpad-style amount field (if not pre-filled from instalment selection)
- Shows: "Minimum: ₹4,200 | Maximum: ₹12,400 | Late fee: ₹240 included if overdue"

#### Collect Payment Button
- Full width, 52dp, `navyBlue` bg, white text "Collect ₹X,XXX"
- Tap flow:
  1. ErpConfirmationDialog: "Record ₹X,XXX payment from [Student Name] via [Method]? This will generate Receipt [Number]."
  2. On confirm: `process_fee_payment()` RPC call
  3. Success: Lottie checkmark animation. Bottom sheet: Receipt preview
  4. [Print Receipt] [Share WhatsApp] [Email] [Done]

#### Recent Transactions (below collection form)
- Card: "Today's Collections" — list of transactions collected today
- Each: [Time] [Student name] [Amount] [Method chip] [Receipt no.]
- [View All →] navigates to fee report

---

## SCREEN 3: VISITOR ENTRY SCREEN
**Route:** `/visitor/entry`
**Access:** L3-L5 with `visitor.write` permission (typically front desk staff)

### Header Info Bar
- "Currently inside: [X] visitors" amber chip if ≥1, teal if 0
- "Pre-approved today: [Y]" link chip → opens pre-approval list

### Entry Form

#### Visitor Basic Info
- [Camera icon + "Take Photo"] — takes visitor photo using camera, shows preview (32px × 32px circular)
- Name: text field (required, autocomplete from history)
- Phone: text field (E.164, triggers autofill from previous visits)
- ID Type: dropdown (Aadhaar / PAN / Passport / DL / Voter ID / Employee ID)
- ID Number: text field (format validates per type)

#### Visit Details
- Purpose: dropdown [Meeting Staff] [Parent Meeting] [Delivery] [Official Visit] [Job Interview] [Other]
- Host (Person to Meet): search field → staff search results → tap to select
- Host Name (manual): if not found in staff search
- Vehicle No.: optional text field

#### Items Section (expandable)
- "Carrying items?" toggle. On expand: text field "List items brought"

#### Badge Number
- Auto-generated sequential badge (daily reset). Shown in bordered navy box: "#042"

#### [Log Entry] Button
- Full width, 52dp, `navyBlue`
- On tap: POST /rest/v1/visitors → success toast "Visitor logged — Badge #042"
- Auto-sends notification to host staff member: "Visitor [Name] is at reception for you"
- Prints badge (if connected printer) via Flutter printing package

---

## SCREEN 4: VISITOR IN-OUT REGISTER & DASHBOARD
**Route:** `/visitor/register`

### Summary Row (top)
- 3 stat chips: [Inside: 12] [Checked Out: 47] [Pre-approved: 3]

### Filter & Search
- Search by name/phone. Date picker chip (default: today). Status filter: [All] [Inside] [Out] [Overstay]

### Visitor List
- Each item: 72dp. Card layout:
  - Left: [Visitor photo circular 40dp] (or initial placeholder)
  - Center: [Name 14sp bold] [Reason 12sp grey] [Host: name 12sp]
  - Right: [Status chip: Inside (teal)/Out (grey)/Overstay (red)] [In time 11sp] [Out time or "—"]
- Tap item: expand to show full details + [Check Out] button (if inside)

### Check Out Flow
- Tap [Check Out]: confirmation prompt → PATCH visitor status to "out", records out_time
- Overstay alert: if visitor has been inside >4 hours, status chip = "Overstay" red. AI flagged in principal alerts.

---

## SCREEN 5: OMR AI GRADING SCREEN
**Route:** `/exam/omr-grading`
**Access:** L3-L5 with `exam.grade` permission

### Setup Panel
- [Select Exam] dropdown → exam schedule
- [Select Subject] → subjects in that exam
- [Total Questions]: number (matches answer key)
- [Answer Key Entry]: grid of correct answers (A/B/C/D per question) — one-time setup

### Scan Interface
- Center: Camera viewfinder with OMR sheet guide overlay (white rectangular frame, corner markers)
- Instructions: "Align OMR sheet within the frame — auto-captures when stable"
- Auto-capture: when sheet detected stable for 1.5 seconds
- Manual: [📷 Capture] button

### After Capture
- Loading: "🤖 AI grading..." Lottie animation (2-3 seconds, Edge Function call)
- Result Card:
  - [Student name if detected] [Roll number if detected]
  - Score: "34/50 — 68%" large in `royalBlue`
  - Grade chip: calculated per grade scale
  - Bubble sheet overlay: correct = teal fill, wrong = red fill, unanswered = grey
  - Any ambiguous bubbles: highlighted amber with warning icon
- [Confirm & Save] | [Rescan] | [Manual Override]
- Manual Override: toggles to allow changing individual answers

### Batch Progress
- Bottom: "Scanned: 24 / 42 students" progress bar. [View All Results] when done.

---

## SCREEN 6: PRINT EXAM PAPER SCREEN
**Route:** `/exam/print-paper`
**Access:** L3-L5

### Paper Composition (Left Panel on Desktop, Full Screen on Mobile)
- [Select Question Bank] → subject filter
- Question List: searchable, filterable by type/difficulty/topic/Bloom's level
- Each question: checkbox + [Marks] + difficulty badge + question preview 2-line
- [+ Add Selected to Paper] button

### Paper Preview (Right Panel Desktop / Tab on Mobile)
- Live preview of composed paper
- Sections: [Section A: MCQs] [Section B: Short Answers] [Section C: Long Answers]
- Drag-reorder within sections
- Total marks indicator (running count, compare to target)
- [AI Suggest Questions] button → fills remaining marks quota using AI from question bank

### Print Settings Panel (bottom sheet)
- Paper Size: [A4] [A3] dropdown
- Copies: number input
- Include: [Answer Key] toggle | [Instructions] toggle | [School letterhead] toggle
- Set Paper: [Marks per section] [Time duration] [Student name field on header]

### [Generate & Print] Button
- Calls `/functions/v1/generate_pdf` Edge Function
- Returns PDF → Flutter `printing` package opens print dialog
- Increments `exam_papers.print_count`

---

## SCREEN 7: TRANSPORT & GPS TRACKING SCREEN
**Route:** `/transport/live-map`
**Access:** All roles (filtered by own child's bus for L6)

### Full-Screen OSM Map (flutter_map)
- Base layer: OpenStreetMap tiles
- Map controls: [+/-] zoom, [Location] center on school, [Fit All Buses] button
- School marker: large building icon at school coords (from schools.coords)

### Bus Markers on Map
- Each active vehicle: custom bus icon 40dp × 40dp
- Color: teal if on time, orange if slightly late (>5min), red if very late (>15min)
- Direction: icon rotates based on `vehicle_tracking.heading`
- Tap bus icon: opens Bus Info Bottom Panel

### Bus Info Bottom Panel
- Slides up from bottom (50% screen height). Rounded top corners 20dp. Drag handle.
- [Vehicle No: KA-01-1234] bold 16sp
- [Route: South Campus - Sector 14] 14sp grey
- [Driver: Suresh Yadav] + [📞 Call] icon button
- [Conductor: Ram Singh] + [📞 Call]
- [Speed: 42 km/h] — orange if >60 km/h (speed limit)
- [Students on board: 28/42]
- [Last GPS update: 12s ago] — red if >30s (connectivity issue)
- Below: Student list for this route (scrollable, shows stop + student name)

### Route Selector Chips (horizontal scroll above bottom panel)
- All active routes as chips. Active route: `navyBlue` bg white text. Inactive: white outlined.
- Badge on each chip: student count on that bus
- "All Routes" chip: default. Tap route chip: map zooms to show only that bus and its route line

### Route Polyline
- Drawn in `royalBlue`. Route stops: circular markers (white fill, `royalBlue` border)
- Tap stop marker: ETA popup card (200dp wide):
  - [Stop: Sector 14 Gate] [Current ETA: 8 min] [Arrival: ~10:45 AM] [Students at stop: 4]
  - ETA: computed by OSRM (bus current position → stop position). Updates every 30s.

### Parent View (L6 only — simplified)
- Same map but shows ONLY own child's bus
- Bottom card (always visible): [Bus KA-01-1234] [Driver: Suresh Yadav] [ETA to My Stop: 8 min countdown timer]
- Countdown: live animation, updates every 10s
- [🔔 Notify when 2 min away] toggle — enables push notification via Edge Function geofence

### SOS Alert Overlay
- Triggered by driver SOS button (hardware or app)
- Full-screen red semi-transparent overlay
- Red pulsing badge on bus icon
- Alert bar (top): "🚨 SOS from Bus KA-01-1234 — Driver Triggered. Location: [reverse geocoded address] [Time]"
- Buttons: [📞 Call Driver] [📞 Call School] [Dismiss — L3 only]
- All L3+ admins see this realtime via Supabase Realtime

### Speed Alert Toast
- Shows 5 seconds at top of screen when bus >60 km/h
- Amber bg, warning icon, bold: "⚠ Bus KA-01-1234 traveling at 72 km/h (limit: 60 km/h)"
- Created by Edge Function speed check. Logged as incident in DB.

---

## SCREEN 8: PRIVILEGE MANAGEMENT SCREEN
**Route:** `/settings/privileges`
**Access:** L3 (Principal) ONLY — cannot be delegated

### User Selector (top)
- Search field: "Search staff by name, role, or department"
- Results list: [Avatar 36dp] [Name 14sp bold] [Role chip: colored per role] [Dept] [Current privilege count badge: navyBlue]
- Tap user: loads their privilege panel below

### Privilege Summary Card (after user selected)
- White card: [User name + avatar] [Role: Teacher (default)]
- [Active privilege overrides: 3] — blue chips showing what's granted
- [Expiring this week: 1] — amber chip
- [Edit All] button → opens full privilege editor

### Resource Accordion List
- Groups: Academic | Fee & Finance | Attendance | Exam & Marks | HR & Leave | Transport | Visitor | Communication | Reports | Settings
- Each group: expandable card. Group header: `F5F5F5` bg with group name bold
- Inside each group: Permission Toggle Row per resource

### Permission Toggle Row (per resource, height 52dp)
- [Resource name 14sp] [Inherited label grey if using default]
- Right side: 6 icon-toggle buttons (36dp tap area each):
  - [R] Read | [W] Write | [D] Delete | [P] Print | [E] Export | [A] Admin
  - Toggle state: grey = inherited (role default) | blue = explicitly granted | red = explicitly denied
  - Long-press each letter: tooltip shows full name "Read / Write / Delete / Print / Export / Admin"

### Valid Till Date Picker (collapsible section)
- [No expiry] chip (default) | [Set expiry date] chip
- On "Set expiry": inline date picker appears below. Shows amber info card:
  - "This privilege will auto-expire on [date] and revert to role defaults"

### Diff Preview Dialog (before saving)
- Full-screen dialog. Title: "Confirm privilege changes for [Name]"
- Two-column diff table: [Resource] | [Current] | [New]
- Changed rows: amber highlight. Added: green. Removed: red.
- [Reason field: optional text input] 
- [Cancel] | [Save Changes]
- On save: privilege record upserted, audit_log entry created, AI action log entry created

### Privilege Templates (menu or top action)
- Dropdown: [Apply Template]: Exam Coordinator | Finance Observer | Communication Editor | Custom...
- Applying: pre-fills all toggles from template
- Save template: captures current config + name field

### Audit Trail Tab
- Separate tab on same screen. White table.
- Columns: [Date] [Changed by] [Resource] [Old Permission] [New Permission] [Reason]
- Sortable, exportable

### Bulk Apply Options
- Action bar above resource list: [Apply to all class teachers] [Apply to this department] [Apply to all L5 staff]
- Shows: "This will affect X users" before confirmation
- AI logs this as bulk privilege action

---

## SCREEN 9: AI ACTIVITY LOG SCREEN
**Route:** `/ai/activity-log`
**Access:** L3+ only

### Summary Header Card
- Full width, 100dp, `purpleSoft` bg, brain icon 32dp purple
- [🤖 AI Activity Log] | [Today: 14 actions] | [Time saved: ~2.3 hrs] | [Auto-handled: 8] | [Requires review: 2]
- Time saved: each action type has estimated manual time constant

### Module Filter Tabs (horizontal scroll)
- [All] [Attendance] [Fee] [Exam] [HR] [Visitor] [Transport] [Communication] [Scheduled]
- Each tab: count badge of today's actions. Active: `royalBlue` underline.

### Search Bar
- Below tabs. Outlined field. Placeholder: "Search by action, student, staff, or module"
- Real-time filter. Clear ✕ button.

### Timeline List (ListView)
- Each item height: 80dp when collapsed, expands on tap
- Row: [Module icon 20dp colored] [Action title 14sp bold] [Timestamp 12sp grey] [Status chip] [Affected count badge: navyBlue]
- Status chips: Success (greenSoft/green) | Pending (orangeSoft/orange) | Failed (redSoft/red) | Undone (grey/grey) | Cancelled (grey/grey)
- Alternating row bg: white and `F5F5F5`

### Expanded Detail View
- Inline expansion below list item (animated slide down)
- Background: `EEF2FF` soft blue
- Monospace detail text:
  - [User: Principal Sharma]
  - [Screen: Fee Collection]
  - [Function: collect_fee()]
  - [Input: {student_id: abc, amount: 4500, method: upi}]
  - [Result: {receipt_no: DPS-2526-000234}]
  - [Notified: Parent Suresh Kumar — WhatsApp]
  - [Undo window: 47 mins remaining] → or "Undo window expired"
- [Undo button] if undoable (amber outlined, visible for 1 hour after action)

### Undo Flow
- Tap [Undo]: confirmation dialog:
  - "This will reverse the fee payment record #DPS-000234. The receipt will be invalidated. Parent will be notified."
  - [Cancel] | [Undo Action]
- After undo: status chip → "Undone". New undo log entry created. Reverse transaction logged.

### Auto-Actions Section
- Separate section at top: "Automatically handled by AI today"
- Navy header row. Examples: "🤖 Sent fee reminders to 23 parents — 8:00 AM" | "🤖 Updated OKR progress — Health Score computed"

### Export Button (App Bar right)
- [PDF — full log with school letterhead] | [Download CSV] | [Share compressed log]
- Edge Function generates. Filtered by current tab + search query.

---

## SCREEN 10: ANALYTICS DASHBOARD
**Route:** `/analytics`
**Access:** L3+ (L2 sees cross-school, L3 sees own school)

### Tabs
- [Overview] [Attendance] [Fee] [Academic] [Staff] [Custom Report Builder]

### Overview Tab — Key Charts

#### School Health Score Timeline
- LineChart. X-axis: last 12 months. Y-axis: 0-100 score.
- Single line: `royalBlue`. Trend: slope indicator. AI forecast: dashed.
- Reference zones: ≥80 green bg zone, 60-79 amber zone, <60 red zone.

#### Subject Performance Heat Table
- Full width card. Rows: classes. Columns: subjects.
- Cell value: average marks %. Color intensity = performance.
  - Green (>80%) → Amber (60-80%) → Red (<60%)
- Tap cell: student list for that class+subject with individual marks.
- AI: "Science consistently weak across Class X — pattern across 3 exams."

#### Attendance Analytics
- Card. Line chart: school-wide attendance % by day for last 30 days.
- Reference line: 75% minimum (red dashed). Dips highlighted with red dot.
- Bar chart below: class-wise attendance ranking (sorted low to high).
- AI: "Monday attendance is 6% lower than other weekdays — systematic pattern."

#### Fee Analytics Deep Dive
- Card. Bar chart: fee collection vs target per month.
- Line overlay: collection rate % (right Y-axis).
- By category: pie chart. By payment method: donut chart.
- Defaulter trend: line chart showing whether overdue accounts increasing/decreasing.
- AI: "UPI adoption increased from 34% to 67% in 6 months — continue digital push."

#### Dropout Risk Heatmap (Scatter Plot)
- Card. fl_chart ScatterChart.
- X-axis: Attendance %. Y-axis: Average marks %.
- Quadrant lines at 75% each axis. Quadrant colors:
  - Top-right (≥75% both): green bg "Excellent"
  - Bottom-left (<75% both): red bg "High Risk"
  - Others: amber "Monitor"
- Each dot = one student. Tap dot: [Student name] [Attendance %] [Avg marks %] [Risk Score] [Recommended Intervention →]

#### AI Insights Panel (bottom of Overview)
- `purpleSoft` card. AI brain icon 32dp.
- "Top 5 AI Insights for [Month Year]:" — numbered list. Each: [Deep Dive →] button.

### Custom Report Builder Tab
- Split view: [Module selector list left] [Report canvas right]
- Drag modules from left panel: [Attendance data] [Fee data] [Exam data] [Staff data] [Visitor data]
- Drop into report canvas: configure date range, filters, chart type
- [AI suggest chart type] based on data selected
- [Preview] [Download PDF] [Export Excel] [Schedule monthly email]

---

### All 14 Chart Types Spec (fl_chart widget mapping)

| Chart Type | fl_chart Widget | Data Source | Interaction |
|-----------|----------------|-------------|-------------|
| Attendance line trend | LineChart with AnimatedLineChart | attendance_summary GROUP BY date | Tap point: tooltip. Pinch zoom. |
| Class attendance heatmap | Custom GridView with ColoredBox | attendance_summary per class per date | Tap cell: class+date detail. Color scale legend. |
| Fee collection bar | BarChart grouped (target+collected) | fee_transactions + fee_structures | Tap bar: monthly breakdown. AI forecast dashed overlay. |
| Fee method donut | PieChart with hole | fee_transactions GROUP BY payment_method | Tap segment: list of that method's transactions. |
| Exam distribution histogram | BarChart (marks ranges as groups) | student_marks for exam+class+subject | Tap bar: student list in that marks range. Pass/fail threshold line. |
| Student growth line | LineChart multi-line | student_growth per student per month | Select student: highlight their line. Compare: overlay two students. |
| Dropout risk scatter | ScatterChart with quadrant lines | attendance_summary + student_marks joined | Tap dot: student profile card. Color: risk level. |
| Subject radar | RadarChart | AVG marks per subject for student/class | Overlay: current vs previous term (two datasets). |
| Revenue trend (Director) | LineChart + BarChart combo | fee_transactions aggregated by school+month | Zoom to single school. AI forecast from current month. |
| Visitor volume calendar | Custom CalendarHeatmap widget | visitors GROUP BY DATE(in_time) | Tap day: visitor list for that day. Month navigator. |
| Transport punctuality | LineChart with zero reference line | eta_calculations: (actual-estimated) avg per route | Lines per route. Zoom range. AI flags consistently late routes. |
| Payroll cost trend | Stacked bar chart | monthly_payroll aggregate by month | Stack segments: basic, HRA, DA, other. Hover/tap: breakdown. |
| Health clinic visits bar | BarChart grouped by reason | clinic_visits GROUP BY reason+month | Shows seasonal patterns. AI disease cluster detection. |
| AI score trend (Director) | LineChart multi-line | schools.health_score per school over time | One line per school. Below/above group avg zones. |

---

## SCREEN 11: STUDENT PROFILE — Full Screen
**Route:** `/students/:id`
**Access:** L3-L5 full; L6 own child only (some sections restricted)

### Profile Header (Hero — 160dp)
- Background: `navyBlue` gradient. White text.
- [Large avatar 72dp with house-color ring: red/blue/green/yellow]
- [Student name 20sp bold]
- [Class X-A | Roll 12 | Admission No]
- [House chip] [Status chip: Active / On Leave / Alumni]
- [Edit button — L3-L4 only] [More ⋯ menu]

### Quick Stats Row (below header — 4 mini cards)
- [Attendance: 87% teal] [Last Rank: #5 royalBlue] [Fee Status: DUE amber] [Health: Healthy green]
- Tap each: scrolls to respective tab section

### Profile Tabs (horizontal scroll, below stats)
- [Overview] [Academic] [Attendance] [Fees] [Health] [Transport] [Hostel] [Documents] [Activity]
- Active: `navyBlue` underline. Each tab loads lazy.

### Overview Tab
- Personal: DOB, Gender, Blood Group, Nationality, Aadhaar last-4
- Family: Father name+phone+email, Mother name+phone, Guardian if different
- Emergency contacts. Previous school. Sibling links (if any).

### Academic Tab
- Subject-wise current year performance table:
  - Columns: Subject | Unit Test 1 | Midterm | Unit Test 2 | Final | Average | Grade chip
  - Grade chips: A+ (teal), A (green), B (royalBlue), C (orange), D/F (red)
- Below table: Homework completion rate progress bar
- Lesson plan progress per subject
- AI insight card (purpleSoft): "Strong in Mathematics. Science needs attention."

### Attendance Tab
- Circular gauge: overall attendance % (large, center, teal fill)
- Subject-wise table: [Subject] [Total Classes] [Attended] [%] [Status chip: OK/Warning/Danger]
- Monthly trend line chart (last 6 months)
- Leave history table
- AI absence pattern: "Absences cluster on Mondays. Contacted parent: [Date]."

### Fees Tab
- Current year summary: [Total demand] [Paid] [Balance] [Status chip]
- Instalment timeline (each with status)
- Transaction history: date, amount, method, receipt link
- [Pay Fee] button — for parent L6 or accountant L5

### Health Tab
- Blood group, allergies, medical conditions (read-only, editable by nurse L3 with health.view)
- Recent clinic visits (date, reason, treatment, follow-up)
- Vaccination status

### Transport Tab
- Route assigned, bus number, stop name, estimated pickup time
- [View Live Map →] → opens transport screen filtered to this child's bus

### Documents Tab
- [Admission Form] [Transfer Certificate] [Character Certificate] [Marksheet] — download links

---

## SCREEN 12: PAYROLL DASHBOARD
**Route:** `/payroll`
**Access:** L3 only (with payroll.process permission)

### Payroll Dashboard Screen

#### Summary Cards Row
- [Total Salary This Month: ₹X.XL] [Employees: 86] [Processed: 72] [Pending: 14] [On Hold: 2]

#### Payroll Run Card
- White card: "Run Payroll for [Month Year]"
- Status: [Not Started / Processing / Generated / Paid]
- [Generate Payslips →] → calculates all staff salaries, deductions, PF, ESI
- [Verify & Approve →] → principal reviews anomalies
- [Process Bank Transfer →] → **REQUIRES 2FA CONFIRMATION** — calls process_bank_transfer (L3 ONLY)

#### Staff Payslip List
- Each row: [Staff name] [Designation] [Basic] [HRA] [DA] [Deductions] [Net Payable] [Status chip] [Download PDF]
- Tap row: full payslip detail view
- Filter: [Department] [Status] [Amount range]

#### Payroll Cost Trend Chart
- Stacked bar chart: last 12 months. Stacks: Basic, HRA, DA, Other allowances.
- Total line overlay. Tap month: breakdown popup.

#### Device Adaptations
- Mobile: Cards stacked. Summary at top. Payslip list below.
- Tablet: 2-column: left = controls, right = payslip list
- Desktop: 3 panel: nav + payslip list + payslip detail side panel

---

## SCREEN 13: HOSTEL MANAGEMENT
**Route:** `/hostel`
**Access:** L3-L4 with hostel.view permission

### Hostel Dashboard Screen

#### Room Grid (Visual Layout)
- Visual grid of all hostel rooms. Row = floor. Cell = room.
- Room cell colors: `greenSoft` = available, `redSoft` = full, `orangeSoft` = partial (some beds empty)
- Room cell content: [Room No.] [X/Y occupied]
- Tap room: opens room detail (allotted students, beds available)

#### Occupancy Summary
- [Total Rooms: 48] [Occupied: 41] [Available beds: 23] [Total students: 186]

#### Mess Menu Card
- "Today's Menu" — [Breakfast] [Lunch] [Snacks] [Dinner]
- Each meal: list of items. Edit button (L3).
- AI: "Today's lunch menu nutrition score: 72/100. Add protein-rich item."

#### Device Adaptations
- Mobile: Summary cards + room list view (not grid)
- Tablet: Room grid visible (2-column)
- Desktop: Full room grid (all floors visible)

---

## SCREEN 14: HEALTH / MEDICAL MODULE
**Route:** `/health`
**Access:** L3-L4 with health.view permission; Nurse role (L3 with permission) is primary user

### Health Dashboard

#### Today's Clinic Summary
- [Visits today: 8] [Sent home: 2] [Referred to hospital: 0] [On medication: 12]

#### Quick Log Visit (top — action first)
- Student search → select → [Reason dropdown] [Symptoms text] [Action: Medication/Rest/Sent Home/Referred] [Record Visit] button

#### Recent Visits List
- Each: [Student name + class] [Time] [Reason chip] [Action taken chip] [Follow-up needed? badge]

#### Disease Pattern Chart (AI-powered)
- Monthly bar chart by reason category. AI: "Flu cases up 40% this week — possible outbreak. Alert parents."
- Threshold alert card if any category spikes >2 standard deviations

#### Medical Records
- Tab: [Individual Student Search] → shows all health records for that student
- Confidential: only visible to L3 with health.view and the student's parent (L6)

---

## SCREEN 15: LIBRARY MANAGEMENT
**Route:** `/library`
**Access:** L3-L5 with library.view; Librarian role primary user

### Library Dashboard & Catalogue

#### Dashboard Summary
- [Books: 4,200] [Issued: 287] [Overdue: 34] [Available: 3,913]

#### Issue / Return Quick Actions (top — action first)
- Two prominent buttons: [Issue Book] [Return Book]
- Issue flow: [Scan barcode] or [Search title/ISBN] → [Select Student] → [Set return date] → [Issue]
- Return flow: [Scan barcode] or [Enter accession no.] → shows student + due date → [Return]
- Overdue calculation: if return date < today: show late fine (per school config)

#### Catalogue Search
- Search by: Title, Author, ISBN, Subject, Class level
- Filters: [Available only] [Genre] [Class recommended]
- Each result: [Book cover thumbnail] [Title 14sp bold] [Author 12sp grey] [Available copies chip]
- Tap: book detail + issue history + [Issue →] button

#### Overdue List
- Card: "Overdue Returns — [Count]"
- Each: [Student name + class] [Book title] [Overdue by: X days] [Fine: ₹XX] [Reminder sent: badge]
- [Send Reminder] individual | [Send All Reminders] bulk button → AI action log entry

---

## SCREEN 16: COMMUNICATION HUB
**Routes:** `/communication/announcements`, `/communication/events`
**Access:** L3-L5 with comms.send permission

### Announcements & Circulars Screen

#### Create Announcement (action-first — sticky top card)
- Title field, body text area (rich text), attachment picker
- Target audience: [All] [Class(es): multi-select] [Role: Students/Parents/Staff] [Custom list]
- Channels: [Push notification] [SMS] [WhatsApp] [Email] (multi-select toggles)
- Schedule: [Send Now] | [Schedule: date + time picker]
- [Preview] | [Send/Schedule]

#### Announcements List
- Tabs: [Sent] [Scheduled] [Drafts]
- Each item: [Title 14sp bold] [Sent to: count + audience chip] [Date] [Read: X/Y] [Channels chips]
- Tap: view full announcement + delivery stats per channel

### Events & School Calendar Screen

#### Monthly Calendar View
- Full-page calendar. Color-coded events:
  - Holiday: red dot
  - Exam: orange dot
  - Event (cultural): royalBlue dot
  - PTM: purple dot
  - Sports: green dot
- Tap date: shows event cards for that day

#### Add Event (bottom sheet)
- Title, Date range, Event type dropdown, Description, Attachments
- Is Holiday toggle (marks school holiday in academic calendar)
- Is Mandatory toggle (appears as badge in student/parent dashboards)
- [Save Event]

---

## SCREEN 17: ADMISSIONS MODULE
**Route:** `/admissions`
**Access:** L3-L4

### Admissions Dashboard
- [Applications received: 145] [Under review: 47] [Approved: 82] [Rejected: 16]
- Funnel chart: Inquiries → Applied → Under Review → Test Scheduled → Approved → Enrolled

### Application List
- Filter tabs: [New] [Under Review] [Approved] [Waitlisted] [Rejected]
- Each item: [Student name + DOB] [Applied class] [Application date] [Application no.] [Status chip]
- Tap: full application detail

### Application Detail Screen
- Personal info + Academic history + Documents uploaded
- Actions: [Schedule Interview] [Schedule Test] [Approve] [Waitlist] [Reject] [Request more docs]
- AI: "Based on academic record and test score (if available), recommend: Approve"

---

## SCREEN 18: SETTINGS & CONFIGURATION
**Route:** `/settings`
**Access:** L3 for school settings; all roles for personal settings

### Personal Settings
- [Profile photo: change] [Name: edit] [Language: dropdown] [Theme: Light/Dark/System]
- [Notification preferences: per channel toggles]
- [Biometric: enable/disable] [2FA: setup/disable]
- [App version] [Logout] [Delete account]

### School Settings (L3 only)
- [School profile: edit logo, contact, board, timezone]
- [Academic year: manage, create new year, transition]
- [Bell schedule: periods, times]
- [Grade scale: A+/A/B/C/D thresholds]
- [Payment config: UPI VPA, Razorpay keys, late fee settings]
- [SMS/WhatsApp: API keys, sender ID]
- [Feature toggles: enable/disable modules per school plan]
