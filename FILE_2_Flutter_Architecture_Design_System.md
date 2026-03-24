# EduVerse Pro v4.0 — FILE 2: Flutter Architecture & Design System
## Agent Scope: Design System · Folder Structure · State Management · Packages · Routing · Offline
**This agent works ONLY on Flutter core architecture. No backend SQL. No UI screens (those are in FILE 3 & 4).**

---

## PART A: DESIGN SYSTEM — Colors, Typography & Spacing

> Design Language: **"Soft Professional"** — pastel-tinted cards, gentle shadows, clean white surfaces.
> Every color is WCAG AA accessible. Every spacing value follows an 8pt grid.

### A.1 Complete Color Palette

```dart
// lib/core/theme/app_colors.dart
// NEVER hardcode colors. ALWAYS use AppColors.xxx

class AppColors {
  // Primary
  static const navyBlue    = Color(0xFF1A2E5A);  // App bar, headers, primary buttons, nav selected — AAA contrast
  static const royalBlue   = Color(0xFF3F51B5);  // Secondary buttons, links, chart primary line — AA contrast
  static const softBlue    = Color(0xFF5C6BC0);  // Hover states, active chips, focus rings — AA contrast

  // Surface
  static const white       = Color(0xFFFFFFFF);  // Card backgrounds, modal backgrounds, list tiles
  static const softSurface = Color(0xFFFAFAFA);  // Page backgrounds, alternating table rows
  static const mutedSurface= Color(0xFFF5F5F5);  // Disabled backgrounds, shimmer base

  // Accent — Status Colors
  static const teal        = Color(0xFF00695C);  // Attendance present, success states, confirm — AA
  static const tealSoft    = Color(0xFFE0F7FA);  // Success card backgrounds, present chip fill
  static const green       = Color(0xFF2E7D32);  // Paid fee, on-track, submitted, available — AA
  static const greenSoft   = Color(0xFFE8F5E9);  // Green card backgrounds, positive metric tint
  static const purple      = Color(0xFF6A1B9A);  // AI widget, premium features, Director portal — AA
  static const purpleSoft  = Color(0xFFF3E5F5);  // AI bubble background, premium card tint
  static const orange      = Color(0xFFE65100);  // Warnings, late status, pending actions — AA
  static const orangeSoft  = Color(0xFFFFF3E0);  // Warning card backgrounds, late chip fill
  static const red         = Color(0xFFC62828);  // Absent, overdue, critical alerts, delete — AA
  static const redSoft     = Color(0xFFFFEBEE);  // Error backgrounds, absent chip fill, overdue card

  // Text
  static const textPrimary  = Color(0xFF1A1A2E); // All primary text, headings — AAA contrast
  static const textSecondary= Color(0xFF666666); // Subtitles, labels, secondary info — AA
  static const textHint     = Color(0xFF9E9E9E); // Placeholder text, hints, disabled (AA large only)

  // Border
  static const borderDefault= Color(0xFFE0E0E0); // Card borders, dividers, table lines
  static const borderFocus  = Color(0xFF3F51B5); // Input focus border, selected item border

  // AI
  static const aiBubble     = Color(0xFF6A1B9A); // AI FAB gradient start, AI message indicator

  // Gold
  static const goldAccent   = Color(0xFFD97706); // Director portal gold accents, star ratings — AA

  // Chart Colors (not for text)
  static const chartBlue    = Color(0xFF1976D2); // Chart line 1, bar chart primary
  static const chartGreen   = Color(0xFF43A047); // Chart line 2, positive trend
  static const chartOrange  = Color(0xFFF57C00); // Chart line 3, warning trend
  static const chartPurple  = Color(0xFF7B1FA2); // Chart line 4, AI forecast
}
```

### A.2 Typography Scale

```dart
// lib/core/theme/app_text_styles.dart
// Fonts: Plus Jakarta Sans (display), DM Sans (body), JetBrains Mono (code)

class AppTextStyles {
  // Display
  static const displayLarge  = TextStyle(fontFamily: 'PlusJakartaSans', fontSize: 32, fontWeight: FontWeight.w700, height: 1.2);  // Hero numbers on Director board
  static const displayMedium = TextStyle(fontFamily: 'PlusJakartaSans', fontSize: 26, fontWeight: FontWeight.w700, height: 1.3);  // Dashboard KPI values, large metric cards
  static const headlineLarge = TextStyle(fontFamily: 'PlusJakartaSans', fontSize: 22, fontWeight: FontWeight.w600, height: 1.3);  // Screen titles, section headings, modal titles
  static const headlineMedium= TextStyle(fontFamily: 'DMSans',          fontSize: 18, fontWeight: FontWeight.w600, height: 1.4);  // Card titles, sub-section headers
  static const titleLarge    = TextStyle(fontFamily: 'DMSans',          fontSize: 16, fontWeight: FontWeight.w600, height: 1.5);  // List item primary, form labels, chip text
  static const titleMedium   = TextStyle(fontFamily: 'DMSans',          fontSize: 14, fontWeight: FontWeight.w500, height: 1.5);  // Sub-labels, badge text, secondary card info
  static const bodyLarge     = TextStyle(fontFamily: 'DMSans',          fontSize: 16, fontWeight: FontWeight.w400, height: 1.6);  // Main body text, descriptions, form input
  static const bodyMedium    = TextStyle(fontFamily: 'DMSans',          fontSize: 14, fontWeight: FontWeight.w400, height: 1.6);  // Secondary body, list subtitles, notifications
  static const bodySmall     = TextStyle(fontFamily: 'DMSans',          fontSize: 12, fontWeight: FontWeight.w400, height: 1.5);  // Timestamps, footnotes, helper text, chart labels
  static const labelLarge    = TextStyle(fontFamily: 'DMSans',          fontSize: 14, fontWeight: FontWeight.w600, height: 1.0);  // Button text, prominent labels
  static const labelSmall    = TextStyle(fontFamily: 'DMSans',          fontSize: 11, fontWeight: FontWeight.w500, height: 1.3);  // Status chips, mini badges, compact info pills
  static const codeStyle     = TextStyle(fontFamily: 'JetBrainsMono',   fontSize: 13, fontWeight: FontWeight.w400, height: 1.6);  // API keys, SQL queries, error codes
}
```

### A.3 Spacing, Elevation & Border Radius Scale

```dart
// lib/core/theme/app_spacing.dart
class AppSpacing {
  static const xs   = 4.0;   // Icon-to-text gap, chip internal padding
  static const sm   = 8.0;   // List item vertical padding, small card padding
  static const md   = 16.0;  // Standard card padding, form field spacing
  static const lg   = 24.0;  // Screen horizontal padding, section spacing
  static const xl   = 32.0;  // Large card padding, modal padding
  static const xxl  = 48.0;  // Hero section padding, login screen spacing
}

// lib/core/theme/app_radius.dart
class AppRadius {
  static const xs   = Radius.circular(4);   // Tags, tight badges
  static const sm   = Radius.circular(8);   // Chips, small cards, table cells
  static const md   = Radius.circular(12);  // Standard cards, buttons, dialogs
  static const lg   = Radius.circular(16);  // Large cards, bottom sheets, feature cards
  static const xl   = Radius.circular(24);  // Hero cards, modal bottom sheet top corners
  static const full = Radius.circular(999); // Circular avatars, pill buttons, FAB
}

// lib/core/theme/app_elevation.dart
class AppElevation {
  // elevation.card: 2dp + BoxShadow(blurRadius:8, offset:Offset(0,2), color:black12)
  static const card  = 2.0;  // Standard cards on white surface
  // elevation.float: 8dp + blurRadius:24
  static const float = 8.0;  // FAB, floating panels, AI widget
  // elevation.modal: 16dp + blurRadius:40
  static const modal = 16.0; // Dialogs, bottom sheets, dropdowns
}
```

---

## PART B: RESPONSIVE BREAKPOINTS — Exact Screen Dimensions

```dart
// lib/core/responsive/breakpoints.dart
class Breakpoints {
  static const phoneXS  = 360.0;   // < 360dp  — Galaxy A03, iPhone SE 1st. 1 col, 12dp h-pad. Bottom nav 4 items.
  static const phoneS   = 360.0;   // 360-399dp — iPhone 12 mini, Moto G. 1 col, 16dp h-pad. Bottom nav 5.
  static const phoneM   = 400.0;   // 400-479dp — iPhone 14, Pixel 7, Samsung S23. 1 col, 16dp h-pad. 1-2 cols (stats).
  static const phoneL   = 480.0;   // 480-599dp — iPhone 14 Plus, Pixel 7 Pro. 1 col, 20dp h-pad. 2 col stats.
  static const tabletS  = 600.0;   // 600-839dp — iPad mini, Galaxy Tab A7. 2 col, 24dp h-pad. Nav Rail collapsed.
  static const tabletM  = 840.0;   // 840-1023dp — iPad 10th gen, Tab S6 Lite. 2 col, list+detail split. Nav Rail expanded.
  static const tabletL  = 1024.0;  // 1024-1199dp — iPad Pro 11", Tab S8. 2-3 col, side panel. Nav Drawer narrow.
  static const desktopS = 1200.0;  // 1200-1439dp — MacBook Air 13". 3 panel: nav(240)+list(320)+detail. Nav Drawer 240dp.
  static const desktopM = 1440.0;  // 1440-1919dp — MacBook Pro 14", Windows. 3 panel wider. Analytics panel. Drawer 260dp.
  static const desktopL = 1920.0;  // 1920dp+ — 4K, iMac. 3 panel, max content 1400dp centered. Drawer 280dp.
}

// Layout helpers
class ResponsiveLayout extends StatelessWidget {
  // Returns appropriate layout based on MediaQuery.of(context).size.width
  // Use LayoutBuilder for nested responsive widgets
}
```

**Navigation Pattern per breakpoint:**
- Phone (< 600dp): BottomNavigationBar (4-5 items)
- Tablet S (600-839dp): NavigationRail (collapsed, icon only)
- Tablet M (840-1023dp): NavigationRail (expanded, icon + label)
- Tablet L (1024-1199dp): NavigationDrawer permanent, narrow
- Desktop (≥ 1200dp): NavigationDrawer permanent, 240-280dp wide

---

## PART C: FLUTTER FOLDER STRUCTURE — Complete with File Purpose

```
eduverse_pro/
├── lib/
│   ├── main.dart                          # App entry, Riverpod ProviderScope, GoRouter
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_constants.dart         # App-wide constants: timeouts, limits, strings
│   │   │   ├── route_names.dart           # All GoRouter route name constants
│   │   │   └── api_endpoints.dart         # Edge Function URLs, Supabase RPC names
│   │   ├── theme/
│   │   │   ├── app_colors.dart            # AppColors class (see Part A.1)
│   │   │   ├── app_text_styles.dart       # AppTextStyles class (see Part A.2)
│   │   │   ├── app_spacing.dart           # AppSpacing, AppRadius, AppElevation
│   │   │   ├── app_theme.dart             # ThemeData (light + dark) using above tokens
│   │   │   └── dark_colors.dart           # Dark mode overrides
│   │   ├── router/
│   │   │   └── app_router.dart            # GoRouter config with all routes + guards
│   │   ├── network/
│   │   │   ├── supabase_client.dart       # Supabase initialization singleton
│   │   │   ├── api_service.dart           # Generic HTTP client for Edge Functions
│   │   │   └── network_service.dart       # Connectivity check, retry logic
│   │   ├── responsive/
│   │   │   ├── breakpoints.dart           # Breakpoint constants (see Part B)
│   │   │   └── responsive_layout.dart     # ResponsiveLayout widget builder
│   │   ├── utils/
│   │   │   ├── date_utils.dart            # TIMESTAMPTZ ↔ local display helpers
│   │   │   ├── currency_utils.dart        # Paise ↔ rupee conversion, INR formatting
│   │   │   ├── validators.dart            # Form field validators
│   │   │   ├── file_utils.dart            # File picker, image compression
│   │   │   └── logger.dart               # Structured logging (debug/info/error)
│   │   └── extensions/
│   │       ├── context_extensions.dart    # context.go(), context.theme, context.l10n
│   │       └── string_extensions.dart     # Capitalize, truncate helpers
│   │
│   ├── data/
│   │   ├── models/                        # Freezed + JSON serializable models — ONE per domain
│   │   │   ├── user_model.dart            # User (role, school_id, institution_id)
│   │   │   ├── student_model.dart         # StudentProfile with nested parent info
│   │   │   ├── staff_model.dart           # StaffProfile with employment details
│   │   │   ├── attendance_model.dart      # AttendanceSession + StudentAttendance
│   │   │   ├── fee_model.dart             # FeeStructure, Ledger, Instalment, Transaction
│   │   │   ├── exam_model.dart            # ExamSchedule, Paper, Marks
│   │   │   ├── visitor_model.dart         # Visitor
│   │   │   ├── transport_model.dart       # Vehicle, VehicleTracking, Route, Stop
│   │   │   ├── notification_model.dart    # Notification
│   │   │   ├── ai_action_log_model.dart   # AiActionLog
│   │   │   ├── leave_model.dart           # LeaveApplication
│   │   │   ├── privilege_model.dart       # Privilege, RolePermission
│   │   │   ├── hostel_model.dart          # HostelRoom, RoomAllotment
│   │   │   ├── health_model.dart          # ClinicVisit, MedicalRecord
│   │   │   ├── library_model.dart         # Book, BookIssue
│   │   │   ├── inventory_model.dart       # InventoryItem, StockTransaction
│   │   │   ├── payroll_model.dart         # PayrollRun, StaffPayslip
│   │   │   ├── communication_model.dart   # Announcement, Event
│   │   │   ├── analytics_model.dart       # Dashboard KPIs, chart data models
│   │   │   ├── okr_model.dart             # OKR, HealthScore
│   │   │   └── appraisal_model.dart       # StaffAppraisal, CPDRecord
│   │   │
│   │   ├── repositories/                  # ONE repo per domain — wraps Supabase calls
│   │   │   ├── auth_repository.dart       # login, logout, OTP, biometric, refresh
│   │   │   ├── user_repository.dart       # get/update user, FCM token registration
│   │   │   ├── student_repository.dart    # CRUD student, search, face embedding
│   │   │   ├── staff_repository.dart      # CRUD staff, directory
│   │   │   ├── attendance_repository.dart # Sessions, mark, summary, reports
│   │   │   ├── fee_repository.dart        # Ledger, process payment, transactions
│   │   │   ├── exam_repository.dart       # Schedules, papers, marks, OMR
│   │   │   ├── visitor_repository.dart    # Log, check-out, register
│   │   │   ├── transport_repository.dart  # Vehicles, live tracking, routes, ETA
│   │   │   ├── ai_repository.dart         # AI chat, function execution, log retrieval
│   │   │   ├── notification_repository.dart # Get, mark read, send
│   │   │   ├── leave_repository.dart      # Apply, approve, reject
│   │   │   ├── privilege_repository.dart  # Get, set privileges, bulk apply
│   │   │   ├── hostel_repository.dart     # Rooms, allotments, mess menu
│   │   │   ├── health_repository.dart     # Clinic visits, records
│   │   │   ├── library_repository.dart    # Books, issues, returns, ISBN lookup
│   │   │   ├── inventory_repository.dart  # Items, stock, barcode scan
│   │   │   ├── payroll_repository.dart    # Run payroll, payslips, bank transfer
│   │   │   ├── communication_repository.dart # Announcements, events, bulk SMS
│   │   │   └── analytics_repository.dart  # Dashboard data, chart data, BI reports
│   │   │
│   │   └── local/                         # Isar offline cache
│   │       ├── isar_service.dart          # Isar DB initialization
│   │       ├── isar_schemas/              # Isar schema classes for each cached entity
│   │       ├── pending_write_queue.dart   # Offline write queue (synced on reconnect)
│   │       └── sync_manager.dart          # SyncManager: flush queue on connectivity restored
│   │
│   ├── features/                          # Feature-based organization — one folder per module
│   │   ├── auth/
│   │   │   ├── providers/auth_provider.dart
│   │   │   └── screens/                   # login, otp_verify, biometric_setup, role_select
│   │   ├── dashboard/
│   │   │   ├── providers/dashboard_provider.dart
│   │   │   └── screens/                   # principal_dashboard, teacher_dashboard, director_board, student_dashboard
│   │   ├── attendance/
│   │   │   ├── providers/attendance_provider.dart
│   │   │   └── screens/                   # mark_attendance, attendance_report, face_scan
│   │   ├── fee/
│   │   │   ├── providers/fee_provider.dart
│   │   │   └── screens/                   # fee_collection, fee_ledger, fee_report, upi_payment
│   │   ├── exam/
│   │   │   ├── providers/exam_provider.dart
│   │   │   └── screens/                   # exam_schedule, omr_grading, print_paper, marks_entry, report_card
│   │   ├── visitor/
│   │   │   ├── providers/visitor_provider.dart
│   │   │   └── screens/                   # visitor_entry, visitor_register, pre_approval
│   │   ├── transport/
│   │   │   ├── providers/transport_provider.dart
│   │   │   └── screens/                   # live_map, route_management, vehicle_list
│   │   ├── students/
│   │   │   ├── providers/student_provider.dart
│   │   │   └── screens/                   # student_list, student_profile, add_student, admissions
│   │   ├── staff/
│   │   │   ├── providers/staff_provider.dart
│   │   │   └── screens/                   # staff_directory, leave_management, appraisal
│   │   ├── hostel/
│   │   │   └── screens/                   # hostel_dashboard, room_grid, mess_menu
│   │   ├── health/
│   │   │   └── screens/                   # health_dashboard, clinic_visit, medical_record
│   │   ├── library/
│   │   │   └── screens/                   # library_dashboard, catalogue, issue_return
│   │   ├── inventory/
│   │   │   └── screens/                   # inventory_dashboard, item_list, stock_entry
│   │   ├── payroll/
│   │   │   └── screens/                   # payroll_dashboard, run_payroll, payslips
│   │   ├── communication/
│   │   │   └── screens/                   # announcements, events, school_calendar, bulk_sms
│   │   ├── analytics/
│   │   │   └── screens/                   # analytics_dashboard, custom_report_builder
│   │   ├── ai/
│   │   │   ├── widgets/
│   │   │   │   ├── ai_fab_button.dart     # Purple floating AI button — present on EVERY screen
│   │   │   │   ├── ai_chat_widget.dart    # Full chat interface — expandable bottom sheet
│   │   │   │   └── erp_confirmation_dialog.dart # MANDATORY before any AI write operation
│   │   │   └── providers/ai_provider.dart
│   │   ├── privilege/
│   │   │   └── screens/                   # privilege_management
│   │   └── settings/
│   │       └── screens/                   # app_settings, school_settings, academic_year
│   │
│   ├── shared/
│   │   ├── widgets/
│   │   │   ├── erp_card.dart              # Standard card with shadow — used everywhere
│   │   │   ├── erp_chip.dart              # Status chip with color coding
│   │   │   ├── erp_button.dart            # Primary/secondary/destructive buttons
│   │   │   ├── erp_text_field.dart        # Standard form field with validation
│   │   │   ├── erp_avatar.dart            # 36dp/48dp/72dp avatar with fallback initials
│   │   │   ├── erp_search_bar.dart        # Live search with debounce
│   │   │   ├── erp_empty_state.dart       # Empty list illustration + message
│   │   │   ├── erp_error_widget.dart      # Error state with retry button
│   │   │   ├── erp_shimmer.dart           # Loading skeleton using Shimmer package
│   │   │   ├── erp_data_table.dart        # Responsive sortable data table
│   │   │   ├── erp_bottom_sheet.dart      # Draggable bottom sheet wrapper
│   │   │   └── erp_export_menu.dart       # PDF/CSV/Excel export dropdown
│   │   └── providers/
│   │       ├── auth_state_provider.dart   # Global auth state, role, permissions
│   │       ├── connectivity_provider.dart # Online/offline state
│   │       └── notification_provider.dart # Unread count, notification list
│   │
│   └── l10n/
│       ├── app_en.arb                     # English strings
│       └── app_hi.arb                     # Hindi strings
│
├── supabase/
│   ├── migrations/                        # All SQL migration files (numbered)
│   │   ├── 00001_create_core_tables.sql
│   │   ├── 00002_create_attendance_tables.sql
│   │   ├── 00003_create_fee_tables.sql
│   │   ├── 00004_create_exam_tables.sql
│   │   ├── 00005_create_transport_tables.sql
│   │   ├── 00006_create_module_tables.sql
│   │   ├── 00007_create_ai_tables.sql
│   │   ├── 00008_create_rls_policies.sql
│   │   └── 00009_seed_role_permissions.sql
│   ├── functions/                         # Edge Functions (Deno)
│   │   ├── ai_chat/index.ts               # Gemini AI chat handler
│   │   ├── ai_function_execute/index.ts   # ErpFunctionExecutor bridge
│   │   ├── scan_omr/index.ts              # OMR grading with Gemini Vision
│   │   ├── evaluate_written_answer/index.ts
│   │   ├── upi_collect_request/index.ts
│   │   ├── upi_payment_webhook/index.ts
│   │   ├── razorpay_webhook/index.ts
│   │   ├── send_notification/index.ts     # FCM + SMS + WhatsApp
│   │   ├── generate_pdf/index.ts          # Report card, receipts, documents
│   │   ├── get_eta/index.ts               # OSRM routing
│   │   ├── export_data/index.ts           # CSV/Excel export
│   │   ├── gemini_proxy/index.ts          # API key proxy for prod
│   │   └── nightly_cron/index.ts          # health_score, reminders, OKR update
│   └── config.toml
│
├── assets/
│   ├── fonts/                             # PlusJakartaSans, DMSans, JetBrainsMono
│   ├── images/
│   │   ├── logo.svg
│   │   ├── empty_states/                  # Illustrations for empty screens
│   │   └── onboarding/                    # Onboarding illustrations
│   └── lottie/                            # Lottie animations (loading, success, AI thinking)
│
└── pubspec.yaml
```

---

## PART D: COMPLETE pubspec.yaml — All Packages with Versions

```yaml
name: eduverse_pro
description: EduVerse Pro — AI-Powered School ERP
version: 4.0.0+1
publish_to: none

environment:
  sdk: '>=3.4.0 <4.0.0'
  flutter: '>=3.22.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # Supabase
  supabase_flutter: ^2.5.0
  
  # State Management
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5
  
  # Navigation
  go_router: ^14.2.0
  
  # Data / Models
  freezed_annotation: ^2.4.1
  json_annotation: ^4.9.0
  
  # Local DB (Offline)
  isar: ^3.1.0+1
  isar_flutter_libs: ^3.1.0+1
  path_provider: ^2.1.3
  
  # AI
  google_generative_ai: ^0.4.3        # Gemini AI SDK
  
  # Maps (OpenStreetMap — NEVER Google Maps)
  flutter_map: ^7.0.2
  latlong2: ^0.9.1
  
  # Charts
  fl_chart: ^0.68.0
  
  # Payment
  upi_india: ^3.0.0+1                  # UPI Direct
  razorpay_flutter: ^1.3.6             # Razorpay fallback
  
  # Camera / ML
  camera: ^0.11.0+2
  google_mlkit_face_detection: ^0.11.0 # Face recognition for attendance
  mobile_scanner: ^5.2.3               # Barcode/QR scanner (library, inventory)
  
  # File & Document
  file_picker: ^8.0.3
  image_picker: ^1.1.2
  image: ^4.2.0                        # Image manipulation
  printing: ^5.13.0                    # PDF printing (display only)
  share_plus: ^9.0.0
  
  # Network & Connectivity
  connectivity_plus: ^6.0.3
  http: ^1.2.1
  dio: ^5.4.3+1
  
  # Notifications
  firebase_messaging: ^15.0.4          # FCM push notifications
  firebase_core: ^3.3.0
  flutter_local_notifications: ^17.2.2
  
  # UI & Animation
  shimmer: ^3.0.0                      # Skeleton loading
  lottie: ^3.1.2                       # Lottie animations
  cached_network_image: ^3.3.1
  flutter_svg: ^2.0.10+1
  
  # Utilities
  intl: ^0.19.0                        # Date/time formatting + localization
  uuid: ^4.4.0
  equatable: ^2.0.5
  logger: ^2.4.0
  
  # Biometric
  local_auth: ^2.3.0
  
  # Storage
  flutter_secure_storage: ^9.2.2       # API keys, tokens
  shared_preferences: ^2.3.1           # User preferences

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.11
  freezed: ^2.5.2
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.0
  isar_generator: ^3.1.0+1
  custom_lint: ^0.6.4
  riverpod_lint: ^2.3.10

flutter:
  uses-material-design: true
  generate: true   # Enable l10n generation
  
  assets:
    - assets/images/
    - assets/lottie/
  
  fonts:
    - family: PlusJakartaSans
      fonts:
        - asset: assets/fonts/PlusJakartaSans-Regular.ttf
        - asset: assets/fonts/PlusJakartaSans-SemiBold.ttf
          weight: 600
        - asset: assets/fonts/PlusJakartaSans-Bold.ttf
          weight: 700
    - family: DMSans
      fonts:
        - asset: assets/fonts/DMSans-Regular.ttf
        - asset: assets/fonts/DMSans-Medium.ttf
          weight: 500
        - asset: assets/fonts/DMSans-SemiBold.ttf
          weight: 600
    - family: JetBrainsMono
      fonts:
        - asset: assets/fonts/JetBrainsMono-Regular.ttf
```

---

## PART E: FLUTTER STATE MANAGEMENT — Riverpod Patterns

### E.1 Provider Structure Pattern
```dart
// lib/features/attendance/providers/attendance_provider.dart

// 1. Simple data fetching — use AsyncNotifierProvider
@riverpod
class AttendanceSessionNotifier extends _$AttendanceSessionNotifier {
  @override
  Future<AttendanceSession?> build(String classId, DateTime date) async {
    return await ref.watch(attendanceRepositoryProvider)
        .getSession(classId: classId, date: date);
  }

  Future<void> markAttendance(List<AttendanceRecord> records) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() =>
      ref.read(attendanceRepositoryProvider)
          .markClassAttendance(records: records)
    );
  }
}

// 2. List with filtering — use StateNotifierProvider pattern
@riverpod
class StudentsNotifier extends _$StudentsNotifier {
  @override
  Future<List<StudentProfile>> build(String classId) async {
    return ref.read(studentRepositoryProvider).listStudents(classId: classId);
  }
}

// 3. Simple derived state — use Provider
@riverpod
int absentCount(AbsentCountRef ref, String sessionId) {
  final records = ref.watch(sessionRecordsProvider(sessionId));
  return records.whenData((r) => r.where((s) => s.status == 'absent').length).valueOrNull ?? 0;
}
```

### E.2 Repository Pattern
```dart
// lib/data/repositories/attendance_repository.dart
// NEVER call Supabase client directly from a widget — ALWAYS use repository

@riverpod
AttendanceRepository attendanceRepository(AttendanceRepositoryRef ref) {
  return AttendanceRepository(ref.watch(supabaseClientProvider));
}

class AttendanceRepository {
  final SupabaseClient _supabase;
  AttendanceRepository(this._supabase);

  Future<List<AttendanceSession>> getSessions({
    required String classId, required DateTime date
  }) async {
    // Always filter by school_id (redundant with RLS, but explicit)
    return await _supabase
        .from('attendance_sessions')
        .select()
        .eq('class_id', classId)
        .eq('date', date.toIso8601String().substring(0, 10))
        .withConverter((data) => data.map(AttendanceSession.fromJson).toList());
  }

  Future<Map<String, dynamic>> markClassAttendance({
    required String sessionId,
    required List<AttendanceRecord> records,
  }) async {
    return await _supabase.rpc('mark_class_attendance', params: {
      'p_session_id': sessionId,
      'p_records': records.map((r) => r.toJson()).toList(),
      'p_teacher_id': _supabase.auth.currentUser!.id,
    });
  }
}
```

### E.3 AI Function Executor Pattern
```dart
// lib/features/ai/erp_function_executor.dart
// EVERY AI write operation MUST pass through this — no exceptions

class ErpFunctionExecutor {
  final SupabaseClient _supabase;
  final AiLogRepository _logRepo;

  Future<dynamic> execute({
    required String functionName,
    required Map<String, dynamic> parameters,
    required BuildContext context,
    required String confirmationMessage,
  }) async {
    // Step 1: Authorization check (server-side)
    final auth = await _supabase.rpc('check_ai_request_authorization', params: {
      'p_user_id': _supabase.auth.currentUser!.id,
      'p_function_name': functionName,
      'p_context': parameters,
    });
    
    if (!auth['authorized']) {
      throw UnauthorizedException(auth['reason']);
    }
    
    // Step 2: MANDATORY confirmation dialog before any write
    final confirmed = await showErpConfirmationDialog(
      context: context, message: confirmationMessage
    );
    if (!confirmed) return null;
    
    // Step 3: Execute function via Edge Function
    final result = await _supabase.functions.invoke('ai_function_execute', body: {
      'function_name': functionName,
      'parameters': parameters,
      'confirmed': true,
    });
    
    // Step 4: MANDATORY AI action log
    await _logRepo.log(
      functionName: functionName,
      inputParams: parameters,
      outputResult: result.data,
    );
    
    return result.data;
  }
}
```

---

## PART F: ROUTING — GoRouter Configuration

```dart
// lib/core/router/app_router.dart
// NEVER use Navigator.push(). ALWAYS use context.go() or context.push()

final appRouter = GoRouter(
  initialLocation: '/auth/login',
  redirect: (context, state) {
    final authState = /* ref.read(authStateProvider) */;
    if (!authState.isLoggedIn) return '/auth/login';
    if (!authState.roleSelected) return '/auth/role-select';
    return null; // Allow navigation
  },
  routes: [
    // Auth
    GoRoute(path: '/auth/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/auth/otp', builder: (_, s) => OtpScreen(phone: s.extra as String)),
    GoRoute(path: '/auth/biometric', builder: (_, __) => const BiometricSetupScreen()),
    GoRoute(path: '/auth/role-select', builder: (_, __) => const RoleSelectScreen()),
    
    // Dashboards (role-gated)
    GoRoute(path: '/dashboard', builder: (_, __) => const RoleAwareDashboard()),
    
    // Attendance
    GoRoute(path: '/attendance/mark', builder: (_, s) => MarkAttendanceScreen(classId: s.uri.queryParameters['classId']!)),
    GoRoute(path: '/attendance/report', builder: (_, __) => const AttendanceReportScreen()),
    
    // Fee
    GoRoute(path: '/fee/collection', builder: (_, __) => const FeeCollectionScreen()),
    GoRoute(path: '/fee/ledger/:studentId', builder: (_, s) => FeeLedgerScreen(studentId: s.pathParameters['studentId']!)),
    
    // Students
    GoRoute(path: '/students', builder: (_, __) => const StudentListScreen()),
    GoRoute(path: '/students/:id', builder: (_, s) => StudentProfileScreen(studentId: s.pathParameters['id']!)),
    
    // ... all other routes
    
    // Privilege Management (L3 only)
    GoRoute(path: '/settings/privileges', builder: (_, __) => const PrivilegeManagementScreen()),
    
    // AI Activity Log (L3+ only)
    GoRoute(path: '/ai/activity-log', builder: (_, __) => const AiActivityLogScreen()),
  ],
);
```

---

## PART G: REALTIME SUBSCRIPTIONS — Supabase Channels

```dart
// lib/shared/providers/realtime_provider.dart

@riverpod
class RealtimeSubscriptions extends _$RealtimeSubscriptions {
  @override
  void build() {
    final supabase = ref.watch(supabaseClientProvider);
    final schoolId = ref.watch(authStateProvider).schoolId;

    // 1. Live bus tracking — high frequency
    supabase.channel('vehicle_tracking_$schoolId')
      .onPostgresChanges(
        event: PostgresChangeEvent.insert,
        schema: 'public', table: 'vehicle_tracking',
        filter: PostgresChangeFilter(type: FilterType.eq, column: 'school_id', value: schoolId),
        callback: (payload) => ref.invalidate(liveVehiclesProvider),
      ).subscribe();

    // 2. Visitor alerts — medium frequency
    supabase.channel('visitors_$schoolId')
      .onPostgresChanges(
        event: PostgresChangeEvent.insert,
        schema: 'public', table: 'visitors',
        filter: PostgresChangeFilter(type: FilterType.eq, column: 'school_id', value: schoolId),
        callback: (payload) => ref.invalidate(activeVisitorsProvider),
      ).subscribe();

    // 3. Notifications — personal
    final userId = ref.watch(authStateProvider).userId;
    supabase.channel('notifications_$userId')
      .onPostgresChanges(
        event: PostgresChangeEvent.insert,
        schema: 'public', table: 'notifications',
        filter: PostgresChangeFilter(type: FilterType.eq, column: 'user_id', value: userId),
        callback: (payload) => ref.invalidate(notificationsProvider),
      ).subscribe();
  }
}
```

---

## PART H: OFFLINE MODE — Isar Cache & Write Queue Strategy

```dart
// lib/data/local/sync_manager.dart

class SyncManager {
  final IsarService _isar;
  final ConnectivityService _connectivity;

  void initialize() {
    _connectivity.onConnectivityChanged.listen((isOnline) {
      if (isOnline) _flushWriteQueue();
    });
  }

  Future<void> _flushWriteQueue() async {
    final queue = await _isar.getPendingWrites();
    for (final write in queue) {
      try {
        await _executeWrite(write);
        await _isar.markWriteComplete(write.id);
      } catch (e) {
        await _isar.incrementRetryCount(write.id);
        // Exponential backoff — max 5 retries
      }
    }
  }
}

// What to cache in Isar:
// - Student list per class (7-day TTL)
// - Attendance sessions + records (current day)
// - Fee ledger per student (1-hour TTL)
// - Notifications (unread)
// - Timetable (7-day TTL)
// - User profile

// Offline writes go to PendingWriteQueue — flushed on reconnect
// Critical: attendance marking, fee collection MUST show "pending sync" badge
```

---

## PART I: CRITICAL FLUTTER ARCHITECTURE RULES

```
1.  NEVER hardcode colors — use AppColors.navyBlue, AppColors.royalBlue etc.
2.  NEVER use Navigator.push() — use context.go() or context.push() from GoRouter
3.  NEVER call Supabase client directly from a widget — use repository classes
4.  NEVER create a StatefulWidget if Riverpod state can handle it
5.  EVERY write operation through AI MUST call ErpFunctionExecutor.execute()
6.  EVERY AI write operation MUST show ErpConfirmationDialog before executing
7.  EVERY AI operation MUST call aiLogRepository.log() after execution
8.  EVERY screen MUST have AiFabButton as child of Scaffold (positioned absolute, bottom-right)
9.  ALL maps use flutter_map + OpenStreetMap — NEVER GoogleMapsFlutterPlatform
10. ALL fees/payments: try UPI direct first (upi_india package). Razorpay is fallback
11. Student role (6) CANNOT call: mark_attendance, collect_fee, approve_leave, or any bulk operation
12. ALL timestamps stored as TIMESTAMPTZ UTC — display in local timezone using intl package
13. ALL financial amounts in INR paise (Integer) in calculations — convert to rupees for display
14. Offline writes go to Isar PendingWriteQueue — SyncManager flushes on reconnect
15. PDF generation happens in Edge Function — NOT in Flutter (too heavy for device)
16. Face recognition: ML Kit face detection + pgvector similarity
17. School Health Score computed by stored procedure — NEVER compute in Flutter
18. ALL Supabase queries filter by school_id (RLS also enforces as backup)
19. Director (L2) shows ONLY their institution's schools — never other institutions
20. NEVER use Navigator.push() anywhere — GoRouter only
```

---

## PART J: BUILD PHASE SEQUENCE

### Phase 1 — Foundation (Days 1–7)
1. All `lib/core/` files: constants, theme, router, network services
2. All `lib/data/models/` (freezed): 25+ models matching DB schema
3. All `lib/data/repositories/`: 15+ repos
4. AI Chat Widget (`ai_chat_widget.dart`) — stub functions first
5. Auth screens: login → OTP → biometric setup → role_select

### Phase 2 — Core Dashboards (Days 8–14)
6. Principal Dashboard — `get_principal_dashboard_data()` RPC
7. Teacher Dashboard — `get_teacher_dashboard_data()` RPC
8. Student/Parent Dashboard — student data queries
9. Director Intelligence Board — `get_director_dashboard_data()` RPC

### Phase 3 — Primary Modules (Days 15–28)
10. Attendance: mark screen (3 modes) + reports
11. Fee: collection (all payment methods) + ledger + reports
12. Visitor: entry + in-out dashboard + pre-approvals
13. Transport: OSM live map + route management
14. Staff HR: directory + leave management

### Phase 4 — Academic Modules (Days 29–42)
15. Timetable viewer + AI generator
16. Exam: schedule + paper composer + print
17. OMR AI Grading + Written Answer Evaluation
18. Report Cards: generation + print + publish
19. Question Bank + AI Question Generator
20. Homework/LMS

### Phase 5 — Operational Modules (Days 43–56)
21. Hostel management + room grid + mess menu
22. Health/Medical module
23. Library + ISBN scanner
24. Inventory + barcode scanning
25. Communication hub + bulk SMS + events
26. Payroll processing + bank transfer

### Phase 6 — Intelligence Layer (Days 57–70)
27. Full AI Chat: connect all 85 functions to ErpFunctionExecutor
28. Analytics/BI: all 14 chart types
29. Student Growth Dashboard
30. Admissions module
31. Alumni + Complaints + Surveys
32. Settings + Privilege Management
33. AI Activity Log with undo

### Phase 7 — Polish & Launch (Days 71–84)
34. Offline mode: Isar caching + write queue
35. Notifications: all 21 triggers wired
36. All device layouts: test on 9 breakpoints
37. Accessibility: semantic labels, large text support
38. Performance: Flutter DevTools profiling, RLS query optimization
39. Security audit: RLS review, API key scan, 2FA test
40. Multi-language: Hindi + English (at minimum)
