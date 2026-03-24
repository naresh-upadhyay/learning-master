# EduVerse Pro v4.0 — FILE 5: AI System, RAG Architecture, Notifications & Cross-Cutting Rules
## Agent Scope: AI Functions · RAG Architecture · Gemini Integration · ErpFunctionExecutor · All 85+ AI Functions · Notification Triggers · Inventory · Realtime · Final Rules
**This agent wires AI functions to the UI. Depends on FILE 1 (backend) and FILE 2 (architecture) being complete first.**

---

## PART A: AI SYSTEM OVERVIEW

EduVerse Pro's AI is powered by **Gemini 1.5 Flash (free tier)** via Supabase Edge Functions.

The AI operates in two modes:
1. **Chat Mode:** Natural language conversation. User types/speaks. AI responds with text + optionally calls ERP functions.
2. **Auto Mode:** Scheduled AI actions (cron-based) — fee reminders, OKR updates, health score computation.

**Zero cost strategy:** Gemini Flash free tier (1M tokens/day) covers 98%+ of school usage. Paid upgrade only for very large institutions.

---

## PART B: RAG (Retrieval-Augmented Generation) ARCHITECTURE

RAG prevents AI hallucination by grounding every response in real school data before calling Gemini.

### RAG Pipeline — Every AI Chat Message

```
User Message: "Who are the students with poor attendance this month?"
         ↓
STEP 1: INTENT CLASSIFICATION (lightweight Gemini call)
         Classify into one of: data_query | action_request | general_question | follow_up
         Extract entities: {time_period: "this month", metric: "attendance", threshold: "poor"}
         
STEP 2: CONTEXT RETRIEVAL (Supabase queries — NOT Gemini)
         → Query attendance_summary WHERE school_id=X AND month=current AND percentage < 75
         → Retrieve: [{student_id, name, class, percentage}] — actual DB data
         → Also retrieve: school profile, current academic year, user's role + permissions
         
STEP 3: PERMISSION CHECK
         → check_ai_request_authorization(user_id, "get_attendance_report", context)
         → If unauthorized: return "You don't have permission to view this data"
         
STEP 4: PROMPT CONSTRUCTION
         System: "You are EduVerse AI for [School Name]. Current date: [date]. User: [role].
                  You have access to the following real data: [retrieved_context as JSON]
                  Only answer based on this data. Do not hallucinate."
         User message: [original user message]
         Available functions: [list of functions user is authorized to call]
         
STEP 5: GEMINI API CALL (with function calling enabled)
         model: gemini-1.5-flash
         tools: [ERP function schemas]
         
STEP 6: RESPONSE HANDLING
         If text response: stream to chat UI
         If function_call: → ErpFunctionExecutor.execute() → confirmation dialog → execute
         
STEP 7: AUDIT LOG
         → aiLogRepository.log(user, function, input, output, screen_context)
```

### RAG Context Sources (what gets retrieved per screen)
| Screen Context | Retrieved Data |
|---------------|----------------|
| principal_dashboard | Today's KPIs, priority alerts, health score |
| fee_collection | Student ledger, outstanding balance, instalment schedule |
| attendance (mark mode) | Class list, previous session data, chronic absentees |
| transport_map | Live vehicle positions, route assignments |
| exam/omr | Exam details, answer key, student list |
| student_profile | Full student record (academic, attendance, fee, health) |
| analytics | Latest chart data, school comparisons |
| privilege_management | User's current privileges, role defaults |

---

## PART C: ALL AI FUNCTIONS — Complete List (85+ Functions)

### Format: `function_name(parameters) → returns | authorized_roles | can_undo`

---

### C.1 Attendance Functions
```
mark_attendance(session_id, student_ids[], status) → attendance_records
  Roles: [3,4,5] | Permission: attendance.mark | Undo: YES (within 30 min)

get_attendance_report(school_id, class_id?, from_date, to_date) → report
  Roles: [3,4,5] | Permission: attendance.view | Undo: NO

get_chronic_absentees(school_id, threshold_pct, days) → [student_ids, stats]
  Roles: [3,4,5] | Permission: attendance.view | Undo: NO

send_absence_alerts(student_ids[], message?) → notification_count
  Roles: [3,4,5] | Permission: attendance.mark + comms.send | Undo: YES (within 1 hr)

generate_attendance_certificate(student_id) → PDF_url
  Roles: [3,4] | Permission: attendance.view + reports.print | Undo: NO
```

### C.2 Fee Functions
```
collect_fee(student_id, amount, method, reference?) → {receipt_no, balance}
  Roles: [3,4,5] | Permission: fee.collect | NOT role 6 | Undo: YES (within 1 hr)

check_fee_status(student_id) → {status, balance, overdue_days, late_fee}
  Roles: ALL (L6 own child only) | Undo: NO

send_fee_reminders(student_ids[]?, class_id?) → notification_count
  Roles: [3,4,5] | Permission: fee.view + comms.send | Undo: YES

apply_fee_waiver(student_id, amount, reason) → {new_balance}
  Roles: [3] | Permission: fee.waive | Undo: YES (within 2 hrs)

get_fee_defaulters(school_id, overdue_days?) → [student_list]
  Roles: [3,4,5] | Permission: fee.view | Undo: NO

generate_fee_receipt(transaction_id) → PDF_url
  Roles: [3,4,5] | Permission: fee.view | Undo: NO

generate_fee_report(school_id, from_date, to_date, format?) → CSV/PDF
  Roles: [3,4,5] | Permission: fee.view + reports.export | Undo: NO

process_payroll(school_id, month, year) → {payslips_count, total_amount}
  Roles: [3] ONLY | Permission: payroll.process | Undo: NO (requires bank reversal)

process_bank_transfer(payroll_run_id) → {transfer_ref, amount}
  Roles: [3] ONLY | Permission: payroll.process | 2FA REQUIRED | Undo: NO
```

### C.3 Attendance & Scheduling Functions
```
generate_timetable(class_id, constraints?) → timetable_object
  Roles: [3,4] | Permission: timetable.write | Undo: YES

get_timetable(class_id | teacher_id, week?) → timetable
  Roles: ALL | Undo: NO

detect_timetable_clashes(school_id) → [clash_list]
  Roles: [3,4] | Permission: timetable.view | Undo: NO

assign_substitute(session_id, substitute_teacher_id) → confirmation
  Roles: [3,4] | Permission: hr.substitute | Undo: YES
```

### C.4 Exam & Academic Functions
```
create_exam_schedule(name, type, classes[], dates) → exam_schedule_id
  Roles: [3,4] | Permission: exam.schedule | Undo: YES (before published)

generate_question_paper(subject_id, total_marks, difficulty_mix) → paper_object
  Roles: [3,4,5] | Permission: exam.question_bank | Undo: YES (before print)

grade_omr(exam_subject_id, student_id, image_base64) → {score, grade, bubble_overlay}
  Roles: [3,4,5] | Permission: exam.grade | Undo: YES (within 24 hrs)

evaluate_written_answer(exam_subject_id, student_id, image_urls[]) → {marks, feedback}
  Roles: [3,4,5] | Permission: exam.grade | Undo: YES (within 24 hrs)

enter_marks(exam_subject_id, marks_data[{student_id, marks}]) → success
  Roles: [4,5 own subject] | Permission: exam.marks | Undo: YES (before published)

publish_results(exam_schedule_id) → published_count
  Roles: [3,4] | Permission: exam.publish | Undo: NO

generate_report_card(student_id, exam_schedule_id) → PDF_url
  Roles: [3,4,5] | Permission: exam.report_card | Undo: NO

generate_class_report_cards(class_id, exam_schedule_id) → [PDF_urls]
  Roles: [3,4] | Permission: exam.report_card | Undo: NO

get_student_performance_analysis(student_id) → ai_analysis_text
  Roles: [3,4,5] | L6: own child only | Undo: NO

identify_at_risk_students(class_id?, threshold?) → [student_list + risk_scores]
  Roles: [3,4,5] | Permission: exam.view + attendance.view | Undo: NO

generate_study_plan(student_id, weak_subjects?) → study_plan
  Roles: [6 own only, 3,4,5] | Undo: NO

answer_student_doubt(subject, question_text, grade_level) → explanation
  Roles: [6] | Undo: NO — read-only AI tutoring
```

### C.5 Visitor & Security Functions
```
log_visitor(visitor_data_object) → {visitor_id, badge_number}
  Roles: [3,4,5] | Permission: visitor.write | Undo: YES (within 30 min)

checkout_visitor(visitor_id) → {out_time}
  Roles: [3,4,5] | Permission: visitor.write | Undo: YES (within 15 min)

get_visitor_report(school_id, from_date, to_date) → visitor_list
  Roles: [3,4,5] | Permission: visitor.view | Undo: NO

flag_visitor(visitor_id, reason) → alert_sent
  Roles: [3,4,5] | Permission: visitor.write | Undo: NO

get_active_visitors(school_id) → [visitor_list]
  Roles: [3,4,5] | Permission: visitor.view | Undo: NO
```

### C.6 Transport Functions
```
get_bus_location(vehicle_id | route_id) → {lat, lng, speed, heading, eta}
  Roles: ALL (L6 own route only) | Undo: NO — read-only

get_eta(vehicle_id, stop_id) → {eta_minutes, arrival_time}
  Roles: ALL (L6 own route only) | Undo: NO

trigger_sos(vehicle_id) → {alert_sent_to: principal_ids[]}
  Roles: Driver only (via special endpoint) | Undo: YES (L3 can dismiss)

log_vehicle_incident(vehicle_id, type, description) → incident_id
  Roles: [3,4,5] | Permission: transport.write | Undo: YES

assign_student_to_route(student_id, route_id, stop_id) → confirmation
  Roles: [3,4] | Permission: transport.manage | Undo: YES
```

### C.7 Staff HR Functions
```
apply_leave(staff_id, leave_type, from_date, to_date, reason) → application_id
  Roles: [4,5] for own leave; [3,4] for bulk | Undo: YES (before approval)

approve_leave(application_id) → confirmation
  Roles: [3,4] | Permission: hr.leave.approve | NOT 5,6 | Undo: YES (within 1 hr)

reject_leave(application_id, reason) → confirmation
  Roles: [3,4] | Permission: hr.leave.approve | Undo: YES (within 1 hr)

get_leave_calendar(school_id, from_date, to_date) → calendar_data
  Roles: [3,4,5] | Permission: hr.view | Undo: NO

generate_appraisal(staff_id, academic_year_id) → appraisal_id
  Roles: [3,4] | Permission: hr.appraisal | Undo: YES (before finalized)

get_staff_directory(school_id, department?) → [staff_list]
  Roles: [3,4,5] | Permission: hr.view | Undo: NO
```

### C.8 Communication Functions
```
send_announcement(title, body, target_audience, channels[]) → notification_count
  Roles: [3,4,5] | Permission: comms.send | Undo: NO (cannot unsend; can publish correction)

send_circular(title, body, attachments[]?, classes[]?) → notification_count
  Roles: [3,4,5] | Permission: comms.send | Undo: NO

schedule_announcement(announcement_data, send_at) → scheduled_id
  Roles: [3,4,5] | Permission: comms.send | Undo: YES (before send_at)

send_bulk_sms(message, phone_numbers[]?) → {sent, failed}
  Roles: [3] | Permission: comms.bulk_sms | Undo: NO

create_event(title, date, type, description, is_holiday?) → event_id
  Roles: [3,4] | Permission: comms.events | Undo: YES

notify_parents(student_ids[], message, channels[]) → notification_count
  Roles: [3,4,5] | Permission: comms.send | Undo: YES (within 5 min for push)
```

### C.9 Analytics & Intelligence Functions
```
get_school_summary(school_id, date?) → KPI_object
  Roles: [1,2,3,4]. L5=own class only. L6=own child only. | Undo: NO

get_school_health_score(school_id) → health_score_object
  Roles: [1,2,3,4] | Undo: NO — calls DB function directly

compare_branches(school_ids[], metric, period) → comparison_report
  Roles: [1,2] ONLY | Undo: NO

get_dropout_risk_analysis(school_id, class_id?) → [at_risk_students]
  Roles: [3,4,5] | Permission: analytics.view | Undo: NO

generate_custom_report(config_object) → PDF/CSV_url
  Roles: [3,4] | Permission: reports.export | Undo: NO

update_okr_progress(okr_id) → {current_value, progress_pct}
  Roles: [3,4] for own school; [2] for institution | Undo: NO — auto-computed

get_ai_recommendations(context_type, context_id) → [recommendations]
  Roles: [2,3,4] | Undo: NO
```

### C.10 Settings & Admin Functions
```
grant_privilege(user_id, resource, actions[], valid_till?) → privilege_id
  Roles: [3] ONLY | Permission: admin.privileges | NEVER 4,5,6 | Undo: YES (revoke)

revoke_privilege(privilege_id) → confirmation
  Roles: [3] ONLY | Permission: admin.privileges | Undo: NO

get_audit_log(school_id, filters?) → audit_entries
  Roles: [3] | Permission: admin.audit | Undo: NO

create_academic_year(school_id, name, start_date, end_date) → academic_year_id
  Roles: [3] | Permission: admin.academic_year | Undo: YES (if no data exists for year)
```

### C.11 Hostel, Health, Library, Inventory Functions
```
-- Hostel
allot_room(student_id, room_id, bed_number) → allotment_id
  Roles: [3,4] | Permission: hostel.manage | Undo: YES

update_mess_menu(date, meals_object) → confirmation
  Roles: [3,4] | Permission: hostel.manage | Undo: YES (within 2 hrs before meal)

-- Health
log_clinic_visit(student_id, reason, symptoms, action_taken) → visit_id
  Roles: [3,4,5 with health.view] | Undo: YES (within 1 hr)

send_home_student(student_id, reason, parent_notified) → confirmation
  Roles: [3,4,5 with health.view] | Undo: NO

-- Library
issue_book(book_id, student_id, return_date) → issue_id
  Roles: [3,4,5 with library.manage] | Undo: YES (within 1 hr)

return_book(issue_id, late_fine?) → confirmation
  Roles: [3,4,5 with library.manage] | Undo: YES (within 1 hr)

add_book(isbn, copies, location) → book_id  -- auto-fetches metadata from ISBN API
  Roles: [3,4,5 with library.manage] | Undo: YES

-- Inventory
log_stock_in(item_id, quantity, supplier, invoice_ref) → stock_transaction_id
  Roles: [3,4,5 with inventory.manage] | Undo: YES (within 2 hrs)

log_stock_out(item_id, quantity, issued_to, purpose) → stock_transaction_id
  Roles: [3,4,5 with inventory.manage] | Undo: YES (within 2 hrs)

reorder_stock(item_id) → purchase_request_id  -- creates PO when stock < reorder_point
  Roles: [3,4] | Permission: inventory.purchase | Undo: YES (before approved)
```

---

## PART D: AI FUNCTION EXECUTOR — Edge Function Implementation

```typescript
// supabase/functions/ai_function_execute/index.ts

import { serve } from "https://deno.land/std/http/server.ts"
import { createClient } from "@supabase/supabase-js"
import { GoogleGenerativeAI } from "@google/generative-ai"

serve(async (req) => {
  const { function_name, parameters, confirmed } = await req.json()
  const authHeader = req.headers.get('Authorization')!
  
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!,
    { global: { headers: { Authorization: authHeader } } }
  )
  
  // 1. Get user from JWT
  const { data: { user } } = await supabase.auth.getUser()
  
  // 2. Authorization check (ALWAYS — even if confirmed=true)
  const { data: auth } = await supabase.rpc('check_ai_request_authorization', {
    p_user_id: user.id,
    p_function_name: function_name,
    p_context: parameters
  })
  
  if (!auth.authorized) {
    return new Response(JSON.stringify({ error: auth.reason }), { status: 403 })
  }
  
  // 3. confirmed must be true (Flutter shows dialog, sends confirmed:true only after user approval)
  if (!confirmed) {
    return new Response(JSON.stringify({ error: 'User confirmation required' }), { status: 400 })
  }
  
  // 4. Execute the function
  let result
  switch (function_name) {
    case 'collect_fee':
      const { data } = await supabase.rpc('process_fee_payment', {
        p_student_id: parameters.student_id,
        p_amount: parameters.amount,
        p_method: parameters.method,
        p_ref: parameters.reference,
        p_collected_by: user.id
      })
      result = data
      break
    case 'mark_attendance':
      const { data } = await supabase.rpc('mark_class_attendance', {
        p_session_id: parameters.session_id,
        p_records: parameters.records,
        p_teacher_id: user.id
      })
      result = data
      break
    // ... all 85+ functions
  }
  
  // 5. Log to ai_action_logs
  await supabase.from('ai_action_logs').insert({
    school_id: user.user_metadata.school_id,
    user_id: user.id,
    function_name,
    action_module: getModule(function_name),
    input_params: parameters,
    output_result: result,
    status: 'success',
    is_undoable: isUndoable(function_name),
    undo_deadline: isUndoable(function_name) ? new Date(Date.now() + 3600000) : null,
    estimated_time_saved_sec: getTimeSaved(function_name)
  })
  
  return new Response(JSON.stringify(result), { status: 200 })
})
```

---

## PART E: AI CHAT WIDGET — Complete Specification

```dart
// lib/features/ai/widgets/ai_chat_widget.dart

// The AI chat is a DraggableScrollableSheet (bottom sheet)
// Initial extent: 0.4 (40% screen). Max: 0.95 (95%). Min: 0.15 (header only).

class AiChatWidget extends ConsumerStatefulWidget {
  final String screenContext;  // Current screen — included in every API call
  
  // Layout:
  // ┌─────────────────────────────┐
  // │ Drag handle (grey 40×4dp)   │  ← top center
  // │ "🤖 EduVerse AI" header     │  ← title + close X
  // │ Suggested actions chips     │  ← horizontal scroll, context-aware
  // ├─────────────────────────────┤
  // │                             │
  // │   Message history list      │  ← ChatMessageBubble widgets
  // │   (scrollable)              │
  // │                             │
  // ├─────────────────────────────┤
  // │ [Text input] [Send/Voice]   │  ← sticky bottom input bar
  // └─────────────────────────────┘
}

// Chat message bubbles:
// User: right-aligned, navyBlue bg, white text, 12dp rounded corners
// AI: left-aligned, purpleSoft bg, textPrimary text, AI icon 20dp left
// System/function result: full width, EEF2FF bg, monospace 12sp, collapsible

// Suggested action chips (context-aware, loaded from AI on screen open):
// Example on fee_collection screen:
// ["Show today's defaulters"] ["Send fee reminders"] ["Top payment method?"] ["Generate report"]
// Tap chip: immediately sends as user message

// Function call confirmation flow within chat:
// AI says: "I'll collect ₹4,500 from Rahul Sharma via UPI. Confirm?"
// Shows: ErpConfirmationCard (inline in chat, not separate dialog)
//   [Student: Rahul Sharma | Amount: ₹4,500 | Method: UPI | Receipt: Auto-generated]
//   [Cancel] [Confirm & Execute]
// On confirm: function executes, result shown as success bubble
```

---

## PART F: NOTIFICATION SYSTEM — All 21 Triggers

### F.1 Notification Architecture
```
Channels: Push (FCM) | SMS (Twilio/MSG91) | WhatsApp (WATI/360Dialog) | Email (Resend)
Edge Function: send_notification/index.ts — handles all channels
Trigger: DB trigger → pg_notify → Edge Function | OR direct RPC call
Storage: notifications table per user
FCM token: stored in users.fcm_tokens[] (array for multi-device)
```

### F.2 Complete Notification Triggers
| # | Trigger Event | Recipients | Channels | Priority |
|---|--------------|-----------|---------|---------|
| 1 | Student marked absent | Parents | Push + SMS | High |
| 2 | Student marked late (>2nd time this week) | Parents | Push | Medium |
| 3 | Fee payment received | Student + Parent | Push + WhatsApp | Low |
| 4 | Fee due in 3 days | Parent | Push + SMS | Medium |
| 5 | Fee overdue (day 1) | Parent + Principal | Push + SMS | High |
| 6 | Bus ETA: 2 minutes from stop | Parent | Push | High |
| 7 | SOS triggered from bus | All L3 admins + Principal | Push + SMS | Critical |
| 8 | Bus speed violation | Principal + Driver's HOD | Push | High |
| 9 | Visitor arrives for staff member | Staff member | Push | Medium |
| 10 | Visitor overstay (>4 hours) | Principal + Security | Push | High |
| 11 | Leave application submitted | Approver (HOD/Principal) | Push + Email | Medium |
| 12 | Leave approved/rejected | Applicant | Push | Medium |
| 13 | Exam result published | Students + Parents | Push + SMS | High |
| 14 | New announcement/circular | Target audience | Push | Medium |
| 15 | New homework assigned | Students + Parents | Push | Low |
| 16 | Homework deadline tomorrow | Students | Push | Medium |
| 17 | AI action completed (auto-mode) | Initiating user | Push | Low |
| 18 | AI undo window expiring (15 min remaining) | Initiating user | Push | Low |
| 19 | Book return overdue (library) | Student + Parent | Push + SMS | Medium |
| 20 | Health: Student sent home | Parent | Push + SMS + WhatsApp | Critical |
| 21 | Health: Disease outbreak alert | All parents in school | Push + SMS | Critical |

---

## PART G: INVENTORY & STORE MANAGEMENT

### Inventory Dashboard Screen
**Route:** `/inventory`
**Access:** L3-L5 with inventory.view permission

#### Summary Cards
- [Total Items: 342] [Low Stock: 12] [Out of Stock: 3] [Total Value: ₹4.2L]

#### Item List
- Search + filter: [Category] [Low stock only] [Out of stock]
- Each item: [Item name] [Category chip] [Current stock: X units] [Unit] [Reorder point] [Status chip]
- Status: "OK" (green, >reorder point) | "Low" (orange, ≤reorder point) | "Out" (red, 0)
- Tap item: full item history (stock in/out log)

#### Stock In Form
- [Select Item] (search / barcode scan) → [Quantity] → [Supplier] → [Invoice Reference] → [Date received] → [Log Stock In]
- Barcode scan: mobile_scanner package. Matches against items table.

#### Stock Out Form
- [Select Item] → [Quantity] → [Issued To: staff/class/department] → [Purpose] → [Log Issue]

#### Low Stock Alerts
- Card with red badge: "12 items below reorder point"
- [Generate Purchase Request] → creates PO for all low-stock items
- AI: "Chalk stock will run out in 3 days at current usage rate"

#### Barcode/QR Scanner
- Tap [📷 Scan] on any item field: opens mobile_scanner
- Matches barcode → auto-fills item name + category
- Unknown barcode: prompts to create new item

---

## PART H: SUPABASE REALTIME — All Subscriptions

```dart
// lib/shared/providers/realtime_provider.dart
// Initialize once after auth, using school_id from JWT

// Active Realtime Channels per user role:

// ALL ROLES:
Channel 'notifications_{user_id}'  → notifications table INSERT
  → invalidate notificationsProvider → badge count updates

// L3-L5:
Channel 'visitors_{school_id}'    → visitors table INSERT
  → invalidate activeVisitorsProvider → dashboard alert updates

Channel 'leave_{school_id}'       → leave_applications INSERT
  → invalidate pendingLeaveProvider → pending actions count

// TRANSPORT (all roles, filtered):
Channel 'tracking_{vehicle_id}'   → vehicle_tracking INSERT (high frequency, 10s)
  → update liveVehicleProvider(vehicleId) → map marker moves

// L3 (Principal):
Channel 'ai_logs_{school_id}'     → ai_action_logs INSERT
  → invalidate aiActivityLogProvider → log screen updates live

Channel 'fee_txn_{school_id}'     → fee_transactions INSERT
  → invalidate feeCollectedTodayProvider → KPI card updates

// SOS (all L3+):
Channel 'sos_{school_id}'         → vehicle_tracking WHERE SOS flag
  → triggers full-screen SOS overlay immediately
```

---

## PART I: NEW AI FUNCTIONS ADDED IN v4.0

These are the new AI-powered capabilities introduced in this version:

### I.1 AI Timetable Generator
```
generate_timetable(school_id, constraints)
constraints = {
  periods_per_day: 8,
  working_days: [Mon-Sat],
  avoid_consecutive_same_subject: true,
  teacher_availability: [...],  -- from leave_applications
  room_capacities: [...],
  lab_subjects: [Physics, Chemistry, CS]  -- need lab periods
}
Algorithm: Constraint satisfaction → Gemini optimizes for teacher workload balance
Output: Full timetable for all classes. Auto-detects clashes.
```

### I.2 AI Written Answer Evaluator
```
evaluate_written_answer(exam_subject_id, student_id, image_urls[])
→ Gemini Vision reads handwritten answer sheet
→ Compares against marking scheme (stored in exam_papers.answer_key)
→ Returns: {marks_awarded, max_marks, reasoning, feedback_for_student}
Teacher can override any mark. AI reasoning shown for transparency.
```

### I.3 AI Dropout Risk Predictor
```
identify_at_risk_students(school_id, class_id?)
Algorithm: Combined score = 
  (attendance_pct < 75: -30) + 
  (marks_avg < 50: -25) + 
  (fee_overdue_days > 60: -20) + 
  (homework_completion < 50%: -15) + 
  (parent_engagement_score < 3: -10)
Score < 40: High Risk. Score 40-60: Medium. Score > 60: Low.
AI generates intervention recommendation per student.
```

### I.4 AI Health Disease Cluster Detection
```
detect_disease_cluster(school_id)
Triggered by: nightly_cron Edge Function
Algorithm: 
  If (clinic_visits by reason this week) > (avg same week last 3 years * 2)
  → "Potential outbreak detected: Flu cases 3x above seasonal average"
  → Triggers notification trigger #21: All parents + Health ministry report
```

### I.5 AI Academic Performance Predictor
```
predict_final_performance(student_id)
Uses: Last 3 unit tests + attendance pattern + homework completion rate
Gemini: predicts final exam score range + likely grade
Shows: "Based on current trend, [Student] is likely to score 72-78% in finals (Grade B)"
```

### I.6 AI Smart Fee Reminder
```
generate_smart_reminder(student_id)
Instead of generic: "Your fee of ₹4,500 is due"
AI generates personalized: "Dear [Parent], [Child]'s Q3 fee installment of ₹4,500 is due on [date]. 
Payment history shows you typically pay on time — we appreciate that! This month's invoice includes 
tuition + transport. [Pay Now button]"
Personalization: uses payment history, child's academic standing, and parent language preference
```

---

## PART J: STAFF DIRECTORY SCREEN
**Route:** `/staff`
**Access:** L3-L5

### App Bar
- Title: "Staff Directory"
- Right: [+ Add Staff] (L3-L4 only) | [Export] | [Search icon]

### Filter Bar
- [All] [Teaching] [Non-Teaching] [Admin] — tab filters
- Secondary: [Department dropdown] [Designation dropdown]

### Staff List
- Each item: 72dp. [Avatar 48dp] [Name 14sp bold] [Designation 12sp grey] [Department chip] [Phone 12sp] [Email 12sp]
- Left border 4dp: colored per role (royalBlue=Teacher, teal=Admin, orange=Non-teaching)
- Tap: staff profile detail. Long press: quick call/email actions.

### Staff Profile Detail
- Header: [Large avatar 72dp] [Name 18sp bold] [Designation] [Department chip] [Employee Code]
- Tabs: [Overview] [Timetable] [Leave History] [Appraisal] [CPD Records]
- Overview: DOB, joining date, qualification, experience, contact, bank info (L3 only)
- Timetable tab: weekly timetable (class periods assigned to this teacher)
- Leave History: timeline of all leave applications with status
- Appraisal: current year scores + AI-generated feedback
- CPD Records: training history + certificates

---

## PART K: LEAVE MANAGEMENT SCREEN
**Route:** `/leave`
**Access:** L3-L4 (approval), L4-L5 (apply own)

### Pending Approvals Section (action-first, L3-L4 only)
- Top: "⏳ X leave applications pending" orange card
- List: Each 80dp. [Staff avatar + name] [Leave type chip] [From-To dates] [Days: X] [Reason summary]
- Action: [Approve ✓] [Reject ✗] swipe actions. Or tap to see full detail + action buttons.
- Approve flow: checks substitute arranged? If not, prompt to assign substitute.
- [Approve All with substitute TBD] bulk option.

### Apply Leave Form (L5 self)
- [Leave type: dropdown — Sick/Casual/EL/Maternity/Paternity/LOP]
- [From date] [To date] → shows calculated working days
- [Reason: text field]
- [Attach medical certificate: if sick leave > 2 days]
- [Submit Application] → creates leave_application, notifies HOD/Principal

### Leave Calendar View
- Monthly view. Staff names on days they're on leave. Color per leave type.
- Shows: "3 staff on leave [date]" — help principal plan substitutes

### Leave Balance Card (per staff, own view)
- [Sick: 5/12 used] [Casual: 3/7 used] [EL: 8/15 used] [LOP: 0]
- Progress bars per leave type. Reset at academic year start.

---

## PART L: FINAL CROSS-CUTTING RULES CHECKLIST

This file's agent must implement ALL of the following across every AI feature:

### AI Execution Rules (NEVER skip any step)
```
□ check_ai_request_authorization() called BEFORE every write
□ ErpConfirmationDialog shown BEFORE every write
□ aiLogRepository.log() called AFTER every action (success or failure)
□ is_undoable flag set correctly per function (see Part C)
□ undo_deadline = now() + 1 hour for undoable functions
□ estimated_time_saved_sec set per function type

Time saved constants:
  mark_attendance: 180 sec (3 min manual per class)
  collect_fee: 120 sec
  send_reminder: 60 sec per batch
  generate_report: 300 sec
  grade_omr: 60 sec per sheet
```

### Security Rules
```
□ Student role (6): ONLY read own data. No writes except own fee payment + homework submission
□ Cross-school: Verified in stored procedure AND RLS. Double protection.
□ Director: ONLY own institution's schools. Institution_id check mandatory.
□ 2FA required: process_bank_transfer ONLY
□ All sensitive data (bank account, PAN, Aadhaar): encrypted at rest in DB
□ API keys (Gemini, Razorpay): stored in Supabase Vault, accessed via Edge Function ONLY
□ NEVER expose service_role_key to Flutter client
```

### Data Integrity Rules
```
□ All financial: paise integers in DB, rupees on display
□ All timestamps: TIMESTAMPTZ UTC in DB, intl package for display in local timezone
□ All school queries: include school_id filter (even with RLS)
□ Idempotency keys: required for payment endpoints
□ Offline queue: verify integrity on flush — reject if school_id mismatch on reconnect
□ Face embeddings: VECTOR(128) in pgvector, cosine similarity threshold: 0.85
```

### UI Rules
```
□ AiFabButton: present on EVERY screen (Scaffold body, absolute positioned, bottom-right 24dp)
□ Never hardcode colors — AppColors.xxx always
□ Never Navigator.push() — context.go() / context.push() only
□ Shimmer loading for all data-fetching states (ErpShimmer widget)
□ Empty state: ErpEmptyState widget with illustration
□ Error state: ErpErrorWidget with retry button
□ All text in localized strings (l10n/app_en.arb + app_hi.arb)
□ Semantic labels on all interactive widgets (accessibility)
□ ALL maps: flutter_map + OpenStreetMap tiles only — NEVER Google Maps
```
