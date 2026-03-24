# EduVerse Pro v4.0 — FILE 3: Role-Based Dashboards
## Agent Scope: All Dashboard Screens — Principal · Director · Teacher · Student/Parent
**Build UI only. Use repository classes for data. Follow AppColors, AppSpacing, AppRadius from FILE 2.**
**EVERY dashboard screen MUST have AiFabButton as absolute-positioned Scaffold child.**

---

## DESIGN PHILOSOPHY — Action-First, Role-Aware Intelligence

Every dashboard follows three core principles:

**1. Action-First:** The most urgent action is always the first visual element. No information overload — show what requires attention NOW above what is merely informative.

**2. Role-Aware Intelligence:** Each dashboard is completely different per role. The system knows who you are and shows ONLY what is relevant to your authority level. A teacher never sees finance data. A student never sees staff info.

**3. Contextual AI:** The AI assistant FAB is always visible. It knows which screen you're on (`screen_context`) and proactively suggests actions based on current data — e.g., on attendance screen it suggests "Mark 3 absent students who are chronic absentees."

---

## SCREEN 1: PRINCIPAL DASHBOARD (Role L3)
**Route:** `/dashboard` when role=3
**Data source:** `rpc/get_principal_dashboard_data(school_id, date)` — single RPC call

### Mobile (Phone M — 400dp reference)

#### App Bar
- Height: 64dp. Background: `AppColors.navyBlue`
- Left: School logo 32dp circle avatar (white border 2dp) + School name 16sp bold white + "Good morning, [First Name]" 12sp white 70% opacity
- Right: Bell icon (notification badge: red dot with count) + Avatar 36dp

#### Widget 1: Health Score Banner
- Position: Below app bar, full width. Height: 80dp
- Background: gradient `navyBlue → royalBlue` (horizontal)
- Content: [School Health Score: 78/100] [Grade: B — "Good Performance"] [↑ +3 vs last week]
- Right side: Circular progress indicator 56dp, white strokes on transparent bg
- Tap: navigates to `/analytics/health-score`
- This is ALWAYS the first element — shows overall school pulse at a glance

#### Widget 2: Priority Alerts Rail (AI-powered)
- Position: Below health score. Horizontal scroll. Height: 72dp per card
- Cards: 280dp wide. Background: color per severity:
  - Critical: `redSoft` bg, `red` left border 4dp
  - Warning: `orangeSoft` bg, `orange` left border 4dp
  - Info: `purpleSoft` bg, `purple` left border 4dp (AI suggestion)
- Content: [Icon 20dp] [Alert text 13sp bold] [Sub-text 12sp grey] [Action button 28dp]
- Examples:
  - "🔴 23 students absent today — 3 chronic absentees" → [View List]
  - "🟡 ₹2.8L fee overdue — 47 students" → [Send Reminder]
  - "🤖 Timetable clash detected — Period 4 Science" → [Fix Now]
- Empty state: "✅ All systems normal" green card
- Source: `priority_alerts[]` from dashboard RPC

#### Widget 3: KPI Cards Row (4 cards)
- Position: Below alerts. Horizontal scroll. Height: 100dp per card. Each: 140dp wide
- Card 1: TODAY'S ATTENDANCE
  - Top: "👥 Present Today" 11sp grey
  - Value: "892" 26sp bold `textPrimary`
  - Sub: "of 1,024 students — 87.1%" 12sp
  - Bottom bar: Teal fill 87% of width
- Card 2: FEE COLLECTED TODAY
  - Top: "💰 Collected Today" 11sp grey
  - Value: "₹1.24L" 26sp bold `green`
  - Sub: "↑ 12% vs yesterday" 12sp green
- Card 3: STAFF ON LEAVE
  - Top: "👔 Staff on Leave" 11sp grey
  - Value: "4" 26sp bold (orange if >10% of staff, else textPrimary)
  - Sub: "of 86 staff — 3 substitutes arranged" 12sp
- Card 4: ACTIVE VISITORS
  - Top: "🚪 Active Visitors" 11sp grey
  - Value: "12" 26sp bold
  - Sub: "2 overdue checkout" 12sp (red if >0 overdue)

#### Widget 4: Class Attendance Grid (7-day heatmap)
- Position: Below KPIs. Full width card. Title: "Class Attendance — Last 7 Days"
- Table: Row per class+section. Columns: Mon–Sun + class name
- Cell: 44dp × 32dp. Color by attendance %:
  - ≥90%: `tealSoft` bg, `teal` text
  - 75–90%: `greenSoft` bg, `green` text
  - 60–75%: `orangeSoft` bg, `orange` text
  - <60%: `redSoft` bg, `red` text
- Cell shows: percentage number
- Tap cell: bottom sheet with absent student list for that class+date
- Scroll: horizontal for dates, vertical for classes

#### Widget 5: Fee Monthly Chart
- Position: Below attendance grid. Full width card. Title: "Fee Collection — Last 6 Months"
- Chart: fl_chart BarChart. Grouped bars: [Target (grey)] [Collected (teal)]
- Below bars: collection % label in matching color
- X-axis: Month abbreviations. Y-axis: ₹ in lakhs
- Bottom: [Total Collected: ₹84.2L] [Target: ₹96L] [Collection Rate: 87.7%]
- Tap bar: monthly breakdown bottom sheet

#### Widget 6: Staff on Leave Today
- Position: Below fee chart. Full width card. Title: "Staff on Leave Today"
- List: Each item 64dp. [Avatar 36dp] [Name 14sp bold] [Department 12sp grey] [Leave Type chip] [Substitute: Name 12sp or "Not arranged" red]
- Empty: "✅ All staff present today"

#### Widget 7: Upcoming Events (next 7 days)
- Position: Below staff leaves. Full width card. Title: "Upcoming Events"
- List: Each 52dp. [Date chip: dayOfWeek/date] [Event title 14sp] [Type chip: Holiday/Exam/Event/PTM] [Mandatory badge if is_mandatory]

#### Widget 8: Pending Actions
- Position: Below events. Full width card. Title: "Your Pending Actions"
- Action chips — horizontal scroll:
  - "⏳ 3 Leave Approvals" → tap: go to leave management
  - "⏳ 2 Fee Waivers" → tap: go to fee management
  - "⏳ 1 Privilege Request" → tap: go to privilege management
  - "⏳ 5 Visitor Pre-Approvals" → tap: go to visitor screen

#### AI FAB Button (MANDATORY — every screen)
- Position: Absolute. Bottom-right. 24dp from edge
- Size: 56dp circle
- Background: gradient `purple → royalBlue` (diagonal)
- Icon: AI brain icon 28dp white. Animated pulse ring when AI is processing
- Tap: expands to full AI chat bottom sheet (75% screen height)
- AI chat knows screen_context: "principal_dashboard" and pre-loads relevant suggestions

---

### Tablet / Desktop Adaptations

#### Tablet S (600-839dp) — Nav Rail + 2 Columns
- Bottom nav → NavigationRail (collapsed, left side)
- KPI cards: 2×2 grid instead of horizontal scroll
- Class attendance grid: visible without scroll (wider table)
- Fee chart: shows 12 months instead of 6

#### Tablet M (840-1023dp) — Nav Rail Expanded + Master-Detail
- NavigationRail expanded with labels
- Left panel (360dp): KPIs + Priority Alerts
- Right panel: Attendance grid + Charts
- Both panels scrollable independently

#### Desktop S (1200-1439dp) — 3-Panel Layout
- Nav Drawer permanent (240dp) — all navigation items visible
- Center panel (adaptive): Main dashboard grid
- Right panel (320dp): AI suggestions + Quick Actions + Upcoming Events
- All 4 KPI cards visible in a row without scrolling
- Class attendance: Full table with 30-day view

#### Desktop M/L (1440dp+)
- Max content width: 1400dp centered
- 4-column KPI grid
- Analytics panel appears right of dashboard with live charts
- AI chat: side panel instead of bottom sheet (320dp wide)

---

## SCREEN 2: DIRECTOR / CHAIRMAN INTELLIGENCE BOARD (Role L2)
**Route:** `/dashboard` when role=2
**Data source:** `rpc/get_director_dashboard_data(institution_id, date)` — single RPC

### Key Difference from Principal Dashboard
- Shows ALL schools in the institution, not just one
- Focused on comparative analytics and AI cross-school insights
- Financial overview shows consolidated institution revenue
- Can drill down into any individual school's data

### Widget 1: Intelligence Header
- Full width. Height: 100dp. Background: `navyBlue` gradient with gold accent line (bottom: `goldAccent` 3dp)
- Left: [Institution logo] [Institution name 18sp bold white] ["Director Intelligence Board" 12sp white 70%]
- Right: [Current Date] ["Monitoring X schools" 13sp gold] [AI brain icon animated]

### Widget 2: Multi-School Health Score Comparison
- Card: Full width. Title: "School Health Scores"
- Horizontal bar per school:
  - [School name 13sp] [Colored progress bar: 0-100] [Score number 14sp bold] [Grade chip] [Δ vs last month arrow]
- Color: ≥80 = green, 60-79 = orange, <60 = red
- Sorted: best to worst. Tap school bar: drill into that school's dashboard
- AI insight below: "DPS Agra improved 8pts this month — highest gain in the group"

### Widget 3: Consolidated KPI Cards (4 cards)
- "Total Students: 12,847" "Collection This Month: ₹2.4Cr" "Avg Attendance: 86.3%" "Staff Strength: 642"
- Each card: trends vs last month. Tap: opens school-wise breakdown

### Widget 4: Revenue Comparison Chart
- Line chart + bar chart combo. X-axis: last 6 months. Lines: one per school (different colors)
- Y-axis: ₹ in lakhs. Tap line: isolate that school's trend
- AI forecast: dashed line from current month onward
- Filter chips: [By School] [By Month] [By Category]

### Widget 5: Cross-School Attendance Ranking
- Table: [Rank] [School] [Avg %] [This Week] [Last Week] [Trend]
- Sorted by attendance %. Color-coded trend arrows
- AI: "Kanpur branch Monday attendance 6% lower than rest — investigate"

### Widget 6: AI Intelligence Insights Panel
- Full width card. Background: `purpleSoft`. AI brain icon 32dp purple
- Title: "🤖 Top Insights for [Month Year]"
- List of 5 AI-generated insights with [Deep Dive →] CTA per insight:
  1. "Attendance dip in 3 schools on Mondays — systematic pattern detected"
  2. "Fee collection at DPS Mathura is 94% — highest in group. Their reminder strategy works"
  3. "Science marks improved 8% after new lab sessions at DPS Agra — consider replicating"
  4. "34 students at dropout risk across 4 schools — urgent intervention needed"
  5. "Staff attrition rate rising at DPS Kanpur — 5 resignations this quarter"

### Widget 7: OKR Progress Board
- Full width card. Title: "Institution OKRs — [Quarter]"
- List of OKRs with progress:
  - [Objective title] [Progress bar: baseline → current → target] [% complete] [AI forecast date] [Status chip]
- Color: On track (green), At risk (orange), Behind (red), Achieved (teal checkmark)

### Director-Specific Controls
- [Compare Branches →] button: side-by-side comparison of 2 schools (any metric)
- [Export Intelligence Report] → PDF with all cross-school analytics
- [Set Group OKR] → create institution-level OKR

---

## SCREEN 3: TEACHER DASHBOARD (Role L5)
**Route:** `/dashboard` when role=5
**Data source:** `rpc/get_teacher_dashboard_data(teacher_id, date)`

### Teaching-Focused Layout (Mobile)

#### Widget 1: Today's Timetable (Priority #1 for teacher)
- Full width card. Title: "Today — [Day, Date]"
- Timeline view: Each period as horizontal band (64dp height)
- Period: [Period No.] [Time] [Subject chip: colored] [Class: X-A] [Room: 204] [Status: Upcoming/In Progress/Done]
- Current period: highlighted with `royalBlue` left border 4dp + pulsing dot
- "In Progress" period: [Mark Attendance →] action button (orange, prominent)
- Empty periods: "Free Period" in muted grey

#### Widget 2: Attendance Quick Action (only if period in progress or overdue)
- Bright orange card at top (when action needed). Height: 80dp
- "⚠️ Period 3 attendance not marked — 8B" + [Mark Now →] button
- Disappears when marked

#### Widget 3: My Classes — Attendance Summary
- Card. Title: "My Classes — Attendance This Week"
- Table: [Class] [Subject] [Mon] [Tue] [Wed] [Thu] [Fri] [Avg%]
- Cells: colored by %. Tap: view class attendance detail

#### Widget 4: Homework Pending Review
- Card. Title: "Homework to Review (Pending: X)"
- List: [Subject chip] [Class] [Assigned date] [Submissions: 28/32] [Action: Review →]

#### Widget 5: Upcoming Exams (my subjects)
- Card. Compact list. [Exam name] [Subject] [Date] [Days remaining chip]

#### Widget 6: AI Teaching Assistant Suggestions
- `purpleSoft` card. AI brain icon. Content:
  - "📊 Class 8B Science attendance is 72% this month — below threshold. Consider contacting parents of: Rohit Kumar, Priya Singh, +5 more"
  - "📝 3 students in 9A scored <40% in last unit test. Their weak topic: Organic Chemistry. Suggested resource: [Khan Academy link]"

#### Widgets present but teacher-scoped only
- No financial data visible
- No cross-class admin controls
- Privilege system limits what teacher can edit

---

## SCREEN 4: STUDENT / PARENT DASHBOARD (Role L6)
**Route:** `/dashboard` when role=6
**Data source:** Multiple queries (own student data only — enforced by RLS)

### Activity-First Design (optimized for parents checking quickly)

#### Widget 1: My Child Card (Parent view)
- Hero card: 120dp. Background: `navyBlue` gradient
- [Child photo 56dp] [Child name 18sp bold white] [Class X-A • Roll 12] [School name 12sp white 70%]
- If student (self view): same card but "My Profile"

#### Widget 2: Today's Attendance Status (most important for parents)
- Large status card. Height: 80dp
- Present: `tealSoft` bg, large ✓ icon, "Present Today — checked in 8:02 AM"
- Absent: `redSoft` bg, large ✗ icon, "Marked Absent Today" + [View Reason]
- Not marked yet: `orangeSoft` bg, "Attendance not marked yet"
- Monthly stat below: "This month: 22 present, 2 absent — 91.7%"

#### Widget 3: Fee Status Banner
- Paid up: `greenSoft` bg, "✅ All fees paid for this term"
- Due soon: `orangeSoft` bg, "₹8,500 due — [Due Date]. Pay Now →" [Pay button: navy]
- Overdue: `redSoft` bg, "⚠️ ₹8,500 overdue by 12 days. Late fee: ₹240/day" [Pay Now]

#### Widget 4: Today's Schedule
- Card. Compact timetable for today (read-only, own class only)
- [Period] [Time] [Subject] [Teacher] — no editing controls

#### Widget 5: Homework Pending
- Card. List of pending homework assignments
- [Subject] [Topic] [Due date chip] — color: green if due tomorrow+, orange if due today, red if overdue

#### Widget 6: Recent Exam Results
- Card. Last 3 exam results. [Exam] [Subject] [Score] [Grade chip] [Rank in class if visible]

#### Widget 7: Bus ETA (if transport enrolled)
- Card: `navySoft` bg. [Bus KA-01-1234] [ETA: 12 min — animated countdown] [Route: stops] [🔔 Notify 2 min away toggle]
- Real-time update every 10s via Supabase Realtime

#### Widget 8: Upcoming Events (school calendar)
- Compact list. [Date] [Event] [Type chip: Holiday/Exam/Event]

#### Student-Specific: AI Study Assistant (role 6 allowed)
- `purpleSoft` card. [Ask AI Tutor →] chip buttons:
  - "📚 Generate study plan for my weak subjects"
  - "❓ Ask a doubt about today's lesson"
  - "📊 Show my performance analytics"
- AI CANNOT perform any write operation for role 6 (enforced server-side)

---

## SCREEN 5: STUDENT ACTIVITY GROWTH DASHBOARD
**Route:** `/students/:id/growth`
**Access:** L3-L5 for any student; L6 for own child only

### Charts & Impact Screen Layout

#### Widget 1: Student Header (same as Student Profile — see FILE 4)
#### Widget 2: Growth Score Card
- Hero metric: "Growth Score: 74/100" large circular gauge
- Sub-metrics: Academic (82) + Attendance (88) + Extracurricular (65) + Discipline (90)
- AI: "Strong academic improvement this term. Attendance was the key driver."

#### Widget 3: Academic Progress Line Chart
- fl_chart LineChart multi-line
- X-axis: Last 6 exams (abbreviated names)
- Lines: One per subject (different colors, legend below)
- Y-axis: Marks % (0-100). Reference line: 75% (minimum passing)
- Tap point: tooltip with subject name + marks + grade + rank
- Pinch to zoom. AI forecast: dashed line for next exam

#### Widget 4: Subject Radar Chart
- fl_chart RadarChart
- Axes: All subjects. Values: average performance %
- Two datasets: [This term] vs [Last term] — overlay
- Color: royalBlue for current, grey for previous
- Below: Subject with biggest improvement [Subject: +12%] + Subject needing attention [Subject: -8%]

#### Widget 5: Attendance Calendar Heatmap
- Custom widget: Monthly calendar grid (days as cells)
- Cell colors: `tealSoft` = present, `redSoft` = absent, `orangeSoft` = late, grey = holiday
- Month navigator (left/right arrows). Current month default.
- Summary below: [Present: 22] [Absent: 2] [Late: 1] [Holidays: 5]
- Pattern AI: "Absences cluster on Mondays — 4 out of last 6 Monday absences"

#### Widget 6: Homework Completion Rate
- Card. Bar chart: 4 weeks. Each bar: completion % (submitted/total assigned)
- Color: ≥80% green, 60-80% orange, <60% red
- Avg line. AI: "Completion dropped in week 3 — coincided with cricket match week"

#### Widget 7: Rank Trend
- Small LineChart. X-axis: All exams this year. Y-axis: Rank (inverted — lower is better)
- Color: teal. Shaded area below line. Tooltip: [Exam name] [Rank: 5/42]

#### Widget 8: AI Recommendations
- `purpleSoft` card. Title: "🤖 AI Growth Recommendations"
- Bulleted list (3-5 items):
  - "Focus on Chemistry — avg 58%. Suggest: additional practice problems twice/week"
  - "Attendance on Mondays needs attention — inform parents"
  - "Strong extracurricular performer — nominate for leadership roles"
- [Generate Detailed Report →] button → Edge Function generates PDF

---

## SHARED DASHBOARD COMPONENTS

### AiFabButton Widget
```dart
// lib/features/ai/widgets/ai_fab_button.dart
// Present on EVERY screen — no exceptions

class AiFabButton extends StatefulWidget {
  final String screenContext;  // e.g. 'principal_dashboard', 'fee_collection'
  
  @override
  Widget build(BuildContext context) {
    return Positioned(
      bottom: 24, right: 24,
      child: GestureDetector(
        onTap: () => _openAiChat(context),
        child: Container(
          width: 56, height: 56,
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            gradient: LinearGradient(
              colors: [AppColors.purple, AppColors.royalBlue],
              begin: Alignment.topLeft, end: Alignment.bottomRight,
            ),
            boxShadow: [BoxShadow(blurRadius: 16, color: AppColors.purple.withOpacity(0.4))],
          ),
          child: Icon(Icons.auto_awesome, color: Colors.white, size: 28),
        ),
      ),
    );
  }
}
```

### ErpConfirmationDialog Widget
```dart
// lib/features/ai/widgets/erp_confirmation_dialog.dart
// MANDATORY before any AI write operation

class ErpConfirmationDialog extends StatelessWidget {
  final String title;
  final String message;       // Human-readable description of what will happen
  final String actionLabel;   // e.g. "Mark Attendance", "Collect Fee"
  
  // Returns bool: true = confirmed, false = cancelled
  static Future<bool> show(BuildContext context, {required String message, required String actionLabel}) async {
    return await showDialog<bool>(context: context, builder: (_) => ErpConfirmationDialog(message: message, actionLabel: actionLabel)) ?? false;
  }
}
```

### Role-Aware Dashboard Router
```dart
// lib/features/dashboard/screens/role_aware_dashboard.dart
// Reads role from authStateProvider and routes to correct dashboard

class RoleAwareDashboard extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final role = ref.watch(authStateProvider).role;
    return switch (role) {
      0 || 1 => const SaasAdminDashboard(),
      2 => const DirectorIntelligenceBoard(),
      3 => const PrincipalDashboard(),
      4 => const HodDashboard(),
      5 => const TeacherDashboard(),
      6 => const StudentParentDashboard(),
      _ => const ErrorScreen(),
    };
  }
}
```
