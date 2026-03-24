# EduVerse Pro v4.0 — FILE 1: Backend Infrastructure
## Agent Scope: System Setup · Database · Stored Procedures · REST API · Payments
**This agent works ONLY on backend. No Flutter/UI code. No conflicts with other agents.**

---

## PART A: PROJECT SETUP

### Environment Variables (--dart-define)
```
SUPABASE_URL=https://[project].supabase.co
SUPABASE_ANON_KEY=[anon_key]
GEMINI_API_KEY=[key]                        -- Dev only; prod uses Edge Function proxy
GEMINI_PROXY_URL=https://[project].supabase.co/functions/v1/gemini_proxy
RAZORPAY_KEY_ID=[key]                       -- Optional fallback
OSRM_BASE_URL=https://router.project-osrm.org
NOMINATIM_BASE_URL=https://nominatim.openstreetmap.org
```

### Supabase CLI Setup Commands
```bash
supabase init
supabase start
supabase db reset          # runs all migration files in supabase/migrations/
supabase functions deploy  # deploys all edge functions
```

### Flutter Project Init (run once)
```bash
flutter create --org com.eduverse.pro --project-name eduverse_pro .
flutter pub get
dart run build_runner build --delete-conflicting-outputs   # Riverpod + Freezed codegen
flutterfire configure                                      # Firebase for FCM
```

### Project Info
```
PROJECT NAME:   EduVerse Pro
FLUTTER:        3.22+  |  DART: 3.4+
BUNDLE ID:      com.eduverse.pro
APP NAME:       EduVerse Pro
```

---

## PART B: COMPLETE DATABASE SCHEMA — All Tables with Data Types & Constraints

> **Rules for all tables:**
> - All IDs are UUID v4 (`gen_random_uuid()`)
> - All timestamps are `TIMESTAMPTZ` (UTC storage, local display via `intl` package)
> - String lengths are realistic maximums with buffer
> - All financial amounts stored in **INR paise** (INTEGER) in calculations; convert to rupees for display
> - All queries MUST filter by `school_id` (RLS also enforces as backup)

---

### 2.1 Platform & Identity Tables

#### TABLE: saas_owners
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | Unique SaaS owner ID |
| name | VARCHAR | (200) | NOT NULL | idx_saas_owners_name | Organisation name |
| email | VARCHAR | (320) | NOT NULL | UNIQUE | Login email (RFC 5321 max 320) |
| phone | VARCHAR | (20) | NULL | | E.164 format +919876543210 |
| plan_tier | SMALLINT | DEFAULT 1 | NOT NULL | CHECK (1..5) | 1=Trial 2=Starter 3=Pro 4=Enterprise 5=Custom |
| max_institutions | SMALLINT | DEFAULT 1 | NOT NULL | CHECK (>=1) | Max schools allowed on plan |
| mrr | NUMERIC | (12,2) DEFAULT 0 | NOT NULL | | Monthly Recurring Revenue in INR |
| logo_url | TEXT | | NULL | | Supabase Storage path for logo |
| primary_color | CHAR | (7) DEFAULT #3F51B5 | NOT NULL | CHECK matches #[0-9A-Fa-f]{6} | Hex color for white-label |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | Soft suspend without delete |
| trial_ends_at | TIMESTAMPTZ | NULL | NULL | | NULL = not on trial |
| support_email | VARCHAR | (320) | NULL | | Support contact email |
| config | JSONB | DEFAULT {} | NOT NULL | GIN index | White-label config: fonts, colors, modules |
| created_at | TIMESTAMPTZ | now() | NOT NULL | idx_created_at | Record creation time |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | Last update via trigger |

#### TABLE: institutions
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| saas_owner_id | UUID | | NOT NULL | FK saas_owners(id) | |
| name | VARCHAR | (300) | NOT NULL | idx_institutions_name | Trust/group name |
| code | VARCHAR | (20) | NULL | UNIQUE | Short code e.g. DPS-GRP |
| director_id | UUID | | NULL | FK users(id) | Current director |
| address_line1 | VARCHAR | (500) | NOT NULL | | |
| city | VARCHAR | (100) | NOT NULL | | |
| state | VARCHAR | (100) | NOT NULL | | |
| country | VARCHAR | (60) | DEFAULT India | NOT NULL | |
| phone | VARCHAR | (20) | NOT NULL | | |
| email | VARCHAR | (320) | NOT NULL | | |
| logo_url | TEXT | | NULL | | |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| config | JSONB | DEFAULT {} | NOT NULL | GIN index | Institution-level settings |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: schools
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| institution_id | UUID | | NOT NULL | FK institutions(id) | |
| name | VARCHAR | (300) | NOT NULL | idx_schools_name, idx_schools_institution | Full school name |
| code | VARCHAR | (20) | NULL | UNIQUE per institution | Short code e.g. DPS-AGR |
| principal_id | UUID | | NULL | FK users(id) | Current principal |
| address_line1 | VARCHAR | (500) | NOT NULL | | Street address |
| address_line2 | VARCHAR | (300) | NULL | | Area / locality |
| city | VARCHAR | (100) | NOT NULL | idx_schools_city | |
| state | VARCHAR | (100) | NOT NULL | | |
| pincode | CHAR | (6) | NOT NULL | CHECK (pincode ~ ^[0-9]{6}$) | Indian 6-digit pincode |
| coords | POINT | | NULL | GIST index for geo queries | PostgreSQL POINT(lng, lat) |
| phone | VARCHAR | (20) | NOT NULL | | E.164 school phone |
| email | VARCHAR | (320) | NOT NULL | | School official email |
| board | VARCHAR | (20) | NOT NULL | CHECK IN (CBSE,ICSE,STATE,IB,IGCSE,NIOS) | Affiliation board |
| founded_year | SMALLINT | NULL | NULL | CHECK (1800..2030) | |
| logo_url | TEXT | | NULL | | Supabase Storage path |
| timezone | VARCHAR | (60) | DEFAULT Asia/Kolkata | NOT NULL | IANA timezone string |
| health_score | NUMERIC | (5,2) DEFAULT 0.00 | NOT NULL | CHECK (0..100) | AI composite score |
| health_computed_at | TIMESTAMPTZ | NULL | NULL | | When health_score last computed |
| academic_config | JSONB | DEFAULT {} | NOT NULL | GIN index | Periods per day, bell schedule, grade scale |
| payment_config | JSONB | DEFAULT {} | NOT NULL | GIN index | UPI VPA, Razorpay keys, payment settings |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| is_deleted | BOOLEAN | DEFAULT false | NOT NULL | | Soft delete flag |
| created_by | UUID | | NULL | FK users(id) | L0/L1 who created this school |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: users
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY, FK auth.users(id) | Maps to Supabase Auth user |
| email | VARCHAR | (320) | NOT NULL | UNIQUE, idx_users_email | Login identifier |
| phone | VARCHAR | (20) | NULL | UNIQUE where NOT NULL | E.164, used for OTP login |
| full_name | VARCHAR | (200) | NOT NULL | idx_users_name GIN trgm | Fuzzy search enabled |
| avatar_url | TEXT | | NULL | | Supabase Storage signed URL |
| role | SMALLINT | DEFAULT 6 | NOT NULL | CHECK (0..6) | 0=SaasDev 1=SaasOwner 2=Director 3=Principal 4=HOD 5=Staff 6=Student/Parent |
| school_id | UUID | | NULL | FK schools(id), idx_users_school | NULL for L0/L1 |
| institution_id | UUID | | NULL | FK institutions(id) | NULL for L0; set for L1+ |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| is_deleted | BOOLEAN | DEFAULT false | NOT NULL | | |
| fcm_tokens | TEXT[] | DEFAULT {} | NOT NULL | | Array: multiple devices |
| last_login | TIMESTAMPTZ | NULL | NULL | | |
| locale | VARCHAR | (10) | DEFAULT en | NOT NULL | BCP-47: en, hi, gu, ta, te |
| biometric_enabled | BOOLEAN | DEFAULT false | NOT NULL | | Local biometric auth flag |
| otp_secret | TEXT | | NULL | | For TOTP 2FA |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: academic_years
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| name | VARCHAR | (20) | NOT NULL | | e.g. 2025-26 |
| start_date | DATE | | NOT NULL | | |
| end_date | DATE | | NOT NULL | | |
| is_current | BOOLEAN | DEFAULT false | NOT NULL | One per school CHECK | Only one current per school |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: classes
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id), idx_classes_school | |
| academic_year_id | UUID | | NOT NULL | FK academic_years(id) | |
| name | VARCHAR | (10) | NOT NULL | | e.g. X, XI |
| section | VARCHAR | (5) | NOT NULL | | e.g. A, B |
| class_teacher_id | UUID | | NULL | FK users(id) | |
| room_number | VARCHAR | (20) | NULL | | |
| max_strength | SMALLINT | DEFAULT 40 | NOT NULL | CHECK (1..100) | |
| current_strength | SMALLINT | DEFAULT 0 | NOT NULL | CHECK (>=0) | Updated by trigger |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: student_profiles
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| user_id | UUID | | NOT NULL | FK users(id) UNIQUE | One profile per user |
| school_id | UUID | | NOT NULL | FK schools(id), idx_sp_school | |
| class_id | UUID | | NULL | FK classes(id), idx_sp_class | Current class |
| admission_no | VARCHAR | (30) | NOT NULL | UNIQUE per school | School admission number |
| roll_number | SMALLINT | | NULL | Unique per class | |
| dob | DATE | | NOT NULL | CHECK (age 3..25) | Date of birth |
| gender | VARCHAR | (10) | NOT NULL | CHECK IN (male,female,other) | |
| blood_group | VARCHAR | (5) | NULL | CHECK IN (A+,A-,B+,B-,AB+,AB-,O+,O-) | |
| nationality | VARCHAR | (60) | DEFAULT Indian | NOT NULL | |
| aadhaar_last4 | CHAR | (4) | NULL | CHECK digits only | Last 4 digits for verification |
| house | VARCHAR | (50) | NULL | | House/team name |
| father_name | VARCHAR | (200) | NULL | | |
| father_phone | VARCHAR | (20) | NULL | | |
| father_email | VARCHAR | (320) | NULL | | |
| mother_name | VARCHAR | (200) | NULL | | |
| mother_phone | VARCHAR | (20) | NULL | | |
| guardian_name | VARCHAR | (200) | NULL | | If different from parents |
| guardian_phone | VARCHAR | (20) | NULL | | |
| emergency_contact | JSONB | DEFAULT [] | NOT NULL | | [{name, relation, phone}] |
| address | JSONB | DEFAULT {} | NOT NULL | | {line1, city, state, pincode} |
| previous_school | VARCHAR | (300) | NULL | | |
| face_embedding | VECTOR | (128) | NULL | ivfflat index cosine | pgvector for face recognition |
| photo_url | TEXT | | NULL | | Supabase Storage path |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| is_deleted | BOOLEAN | DEFAULT false | NOT NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: staff_profiles
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| user_id | UUID | | NOT NULL | FK users(id) UNIQUE | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| employee_code | VARCHAR | (30) | NOT NULL | UNIQUE per school | |
| department | VARCHAR | (100) | NOT NULL | | |
| designation | VARCHAR | (100) | NOT NULL | | |
| date_of_joining | DATE | | NOT NULL | | |
| qualification | VARCHAR | (200) | NULL | | |
| experience_years | SMALLINT | DEFAULT 0 | NOT NULL | CHECK (0..50) | |
| bank_account_no | VARCHAR | (30) | NULL | Encrypted at rest | |
| bank_ifsc | VARCHAR | (11) | NULL | CHECK format | |
| pan_number | CHAR | (10) | NULL | Encrypted at rest | |
| salary_grade | VARCHAR | (20) | NULL | | |
| basic_salary | INTEGER | | NULL | In paise | |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |

---

### 2.2 Attendance Tables

#### TABLE: attendance_sessions
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id), idx_as_school | |
| class_id | UUID | | NOT NULL | FK classes(id), idx_as_class | |
| subject_id | UUID | | NULL | FK subjects(id) | NULL = general attendance |
| date | DATE | | NOT NULL | idx_as_date | |
| period_number | SMALLINT | | NULL | CHECK (1..12) | NULL = whole day |
| teacher_id | UUID | | NOT NULL | FK users(id) | Who marked attendance |
| status | VARCHAR | (20) | DEFAULT open | NOT NULL | CHECK IN (open,submitted,locked) |
| total_present | SMALLINT | DEFAULT 0 | NOT NULL | | Updated by trigger |
| total_absent | SMALLINT | DEFAULT 0 | NOT NULL | | Updated by trigger |
| total_late | SMALLINT | DEFAULT 0 | NOT NULL | | |
| marked_at | TIMESTAMPTZ | | NULL | | When submitted |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| UNIQUE | | | | (class_id, date, period_number) | One session per class/period/day |

#### TABLE: student_attendance
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| session_id | UUID | | NOT NULL | FK attendance_sessions(id), idx_sa_session | |
| student_id | UUID | | NOT NULL | FK student_profiles(id), idx_sa_student | |
| status | VARCHAR | (10) | NOT NULL | CHECK IN (present,absent,late,excused,holiday) | |
| reason | VARCHAR | (300) | NULL | | For absent/late |
| marked_by | UUID | | NOT NULL | FK users(id) | |
| face_verified | BOOLEAN | DEFAULT false | NOT NULL | | Was face recognition used? |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| UNIQUE | | | | (session_id, student_id) | One record per student per session |

#### TABLE: attendance_summary
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| student_id | UUID | | NOT NULL | FK student_profiles(id), idx_atsumm_student | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| academic_year_id | UUID | | NOT NULL | FK academic_years(id) | |
| month | SMALLINT | | NOT NULL | CHECK (1..12) | |
| year | SMALLINT | | NOT NULL | | |
| total_working_days | SMALLINT | DEFAULT 0 | NOT NULL | | |
| days_present | SMALLINT | DEFAULT 0 | NOT NULL | | |
| days_absent | SMALLINT | DEFAULT 0 | NOT NULL | | |
| days_late | SMALLINT | DEFAULT 0 | NOT NULL | | |
| percentage | NUMERIC | (5,2) DEFAULT 0 | NOT NULL | CHECK (0..100) | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |
| UNIQUE | | | | (student_id, academic_year_id, month, year) | |

---

### 2.3 Fee Management Tables

#### TABLE: fee_structures
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| academic_year_id | UUID | | NOT NULL | FK academic_years(id) | |
| name | VARCHAR | (200) | NOT NULL | | e.g. Annual Fee Class X 2025-26 |
| class_ids | UUID[] | DEFAULT {} | NOT NULL | | Which classes this applies to |
| components | JSONB | DEFAULT [] | NOT NULL | | [{name, amount, is_optional, due_date}] |
| total_amount | INTEGER | | NOT NULL | | Sum of all components in paise |
| late_fee_per_day | INTEGER | DEFAULT 0 | NOT NULL | | Paise per day late |
| late_fee_grace_days | SMALLINT | DEFAULT 0 | NOT NULL | | |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: student_fee_ledger
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| student_id | UUID | | NOT NULL | FK student_profiles(id) UNIQUE per year | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| academic_year_id | UUID | | NOT NULL | FK academic_years(id) | |
| fee_structure_id | UUID | | NOT NULL | FK fee_structures(id) | |
| gross_amount | INTEGER | NOT NULL | | | Total before discounts (paise) |
| discount_amount | INTEGER | DEFAULT 0 | NOT NULL | | Scholarships + waivers (paise) |
| net_amount | INTEGER | | NOT NULL | | gross - discount (paise) |
| paid_amount | INTEGER | DEFAULT 0 | NOT NULL | | Total paid so far (paise) |
| balance | INTEGER | | NOT NULL | | net - paid (paise) |
| status | VARCHAR | (20) | DEFAULT due | NOT NULL | CHECK IN (due,partial,clear,waived,defaulter) |
| last_payment_date | DATE | | NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| updated_at | TIMESTAMPTZ | now() | NOT NULL | | |

#### TABLE: fee_instalments
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| ledger_id | UUID | | NOT NULL | FK student_fee_ledger(id), idx_fi_ledger | |
| instalment_number | SMALLINT | | NOT NULL | | 1, 2, 3... |
| due_date | DATE | | NOT NULL | | |
| amount | INTEGER | | NOT NULL | | Paise |
| paid_amount | INTEGER | DEFAULT 0 | NOT NULL | | Paise |
| late_fee | INTEGER | DEFAULT 0 | NOT NULL | | Accrued late fee (paise) |
| status | VARCHAR | (20) | DEFAULT unpaid | NOT NULL | CHECK IN (unpaid,partial,paid,waived,overdue) |
| paid_date | DATE | | NULL | | |

#### TABLE: fee_transactions
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id), idx_ft_school | |
| student_id | UUID | | NOT NULL | FK student_profiles(id), idx_ft_student | |
| ledger_id | UUID | | NOT NULL | FK student_fee_ledger(id) | |
| amount | INTEGER | | NOT NULL | | Amount paid in this txn (paise) |
| payment_method | VARCHAR | (30) | | NOT NULL | CHECK IN (upi,cash,cheque,neft,dd,razorpay,bbps) |
| payment_reference | VARCHAR | (200) | | NULL | UPI ref / cheque no / txn ID |
| receipt_number | VARCHAR | (50) | | NOT NULL | UNIQUE — school-year-sequence |
| payment_date | TIMESTAMPTZ | now() | NOT NULL | idx_ft_date | |
| collected_by | UUID | | NOT NULL | FK users(id) | |
| upi_txn_id | VARCHAR | (100) | | NULL | Bank txn ID for UPI |
| gateway | VARCHAR | (30) | | NULL | razorpay / upi_direct / null |
| status | VARCHAR | (20) | DEFAULT success | NOT NULL | CHECK IN (success,pending,failed,reversed) |
| idempotency_key | UUID | | NULL | UNIQUE | Prevents duplicate on retry |
| notes | TEXT | | NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |

---

### 2.4 AI Action Logs & Privileges Tables

#### TABLE: ai_action_logs
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id), idx_aal_school | |
| user_id | UUID | | NOT NULL | FK users(id) | Who triggered the AI action |
| function_name | VARCHAR | (100) | NOT NULL | | e.g. collect_fee, mark_attendance |
| action_module | VARCHAR | (50) | NOT NULL | CHECK IN (attendance,fee,exam,hr,visitor,transport,communication,admin) | |
| input_params | JSONB | DEFAULT {} | NOT NULL | | Parameters passed to function |
| output_result | JSONB | DEFAULT {} | NOT NULL | | Result returned |
| affected_count | INTEGER | DEFAULT 1 | NOT NULL | | Records affected |
| status | VARCHAR | (20) | DEFAULT success | NOT NULL | CHECK IN (success,pending,failed,undone,cancelled) |
| screen_context | VARCHAR | (100) | NULL | | Screen from which AI was triggered |
| session_id | UUID | | NULL | | AI chat session ID |
| is_undoable | BOOLEAN | DEFAULT false | NOT NULL | | Can this be reversed? |
| undo_deadline | TIMESTAMPTZ | | NULL | | NULL or now() + 1 hour |
| undone_at | TIMESTAMPTZ | | NULL | | When undone |
| undone_by | UUID | | NULL | FK users(id) | |
| estimated_time_saved_sec | SMALLINT | DEFAULT 0 | NOT NULL | | Manual equivalent time (seconds) |
| created_at | TIMESTAMPTZ | now() | NOT NULL | idx_aal_created | |

#### TABLE: privileges
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| school_id | UUID | | NOT NULL | FK schools(id) | |
| user_id | UUID | | NOT NULL | FK users(id), idx_priv_user | |
| resource | VARCHAR | (100) | NOT NULL | | e.g. fee.structure, exam.marks |
| actions | VARCHAR[] | DEFAULT {} | NOT NULL | | [read, write, delete, print, export, admin] |
| granted_by | UUID | | NOT NULL | FK users(id) | Must be L3 (Principal) |
| valid_till | DATE | | NULL | | NULL = permanent |
| reason | TEXT | | NULL | | Why granted |
| is_active | BOOLEAN | DEFAULT true | NOT NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | | |
| UNIQUE | | | | (user_id, resource) | One override per user per resource |

#### TABLE: role_permissions (seed data — define defaults per role)
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| role | SMALLINT | | NOT NULL | CHECK (0..6) | |
| resource | VARCHAR | (100) | NOT NULL | | |
| actions | VARCHAR[] | DEFAULT {} | NOT NULL | | Default actions for this role |
| UNIQUE | | | | (role, resource) | |

#### TABLE: audit_log
| Column | Type | Size/Default | Nullable | Constraint/Index | Description |
|--------|------|-------------|----------|-----------------|-------------|
| id | UUID | gen_random_uuid() | NOT NULL | PRIMARY KEY | |
| user_id | UUID | | NOT NULL | FK users(id) | |
| school_id | UUID | | NULL | | |
| action | VARCHAR | (50) | NOT NULL | | INSERT/UPDATE/DELETE/LOGIN/EXPORT |
| table_name | VARCHAR | (80) | NULL | | |
| record_id | UUID | | NULL | | |
| old_value | JSONB | | NULL | | Before state |
| new_value | JSONB | | NULL | | After state |
| ip_address | INET | | NULL | | |
| user_agent | TEXT | | NULL | | |
| created_at | TIMESTAMPTZ | now() | NOT NULL | idx_audit_created | |

---

### 2.5 Additional Core Tables (Summary)

> Detailed schemas follow same pattern as above. All have: id UUID PK, school_id FK, created_at TIMESTAMPTZ.

**Visitors table:** id, school_id, visitor_name VARCHAR(200), phone VARCHAR(20), id_type VARCHAR(20), id_number VARCHAR(50), photo_url TEXT, purpose VARCHAR(200), host_user_id UUID FK, host_name VARCHAR(200), in_time TIMESTAMPTZ, out_time TIMESTAMPTZ, status VARCHAR(20) CHECK IN (in,out,pre_approved,overstay), badge_number VARCHAR(20), vehicle_number VARCHAR(20), items_brought JSONB, created_by UUID FK, created_at TIMESTAMPTZ.

**Notifications table:** id, school_id, user_id FK, title VARCHAR(300), body TEXT, channels TEXT[] (push/sms/whatsapp/email), is_read BOOLEAN DEFAULT false, data JSONB, sent_at TIMESTAMPTZ, created_at TIMESTAMPTZ.

**Leave_applications table:** id, school_id, applicant_id FK users, applicant_type VARCHAR(10) CHECK IN (staff,student), leave_type VARCHAR(30), from_date DATE, to_date DATE, reason TEXT, status VARCHAR(20) CHECK IN (pending,approved,rejected,cancelled), approved_by UUID FK, substitute_id UUID FK, created_at TIMESTAMPTZ.

**Vehicles table:** id, school_id, vehicle_number VARCHAR(20) UNIQUE, vehicle_type VARCHAR(20), driver_id UUID FK, conductor_id UUID FK, capacity SMALLINT, insurance_expiry DATE, fitness_expiry DATE, is_active BOOLEAN, created_at TIMESTAMPTZ.

**Vehicle_tracking table:** id, vehicle_id FK, latitude NUMERIC(10,7), longitude NUMERIC(10,7), speed NUMERIC(5,2), heading SMALLINT, accuracy NUMERIC(6,2), timestamp TIMESTAMPTZ, created_at TIMESTAMPTZ. Partitioned by date. Retention: 90 days.

**Subjects table:** id, school_id, name VARCHAR(100), code VARCHAR(20), department VARCHAR(100), is_practical BOOLEAN DEFAULT false, created_at TIMESTAMPTZ.

**Timetable_slots table:** id, school_id, class_id FK, subject_id FK, teacher_id FK, day_of_week SMALLINT CHECK(1..7), period_number SMALLINT CHECK(1..12), start_time TIME, end_time TIME, room VARCHAR(20), academic_year_id FK, created_at TIMESTAMPTZ.

**Exam_schedules table:** id, school_id, name VARCHAR(200), exam_type VARCHAR(30) CHECK IN (unit_test,midterm,final,practicals,competitive), academic_year_id FK, start_date DATE, end_date DATE, result_published BOOLEAN DEFAULT false, created_at TIMESTAMPTZ.

**Student_marks table:** id, school_id, exam_schedule_id FK, student_id FK, subject_id FK, max_marks SMALLINT, theory_marks NUMERIC(5,2), practical_marks NUMERIC(5,2), total_obtained NUMERIC(5,2), grade VARCHAR(5), is_ai_graded BOOLEAN DEFAULT false, graded_by UUID FK, remarks TEXT, created_at TIMESTAMPTZ. UNIQUE(exam_schedule_id, student_id, subject_id).

---

### 2.6 Additional Module Tables (Sections 33.1–33.3)

#### Exam & Question Bank Tables
```
TABLE: exam_question_banks
  id, school_id, subject_id FK, created_by UUID FK
  question_text TEXT NOT NULL
  question_type VARCHAR(20) CHECK IN (mcq,short_answer,long_answer,fill_blank,true_false,match)
  difficulty VARCHAR(10) CHECK IN (easy,medium,hard)
  marks SMALLINT NOT NULL
  options JSONB DEFAULT []       -- for MCQ: [{text, is_correct}]
  correct_answer TEXT
  bloom_level VARCHAR(20)        -- remember/understand/apply/analyze/evaluate/create
  topic VARCHAR(200)
  chapter VARCHAR(200)
  ai_generated BOOLEAN DEFAULT false
  times_used SMALLINT DEFAULT 0
  created_at TIMESTAMPTZ now()

TABLE: exam_papers
  id, school_id, exam_schedule_id FK, subject_id FK
  title VARCHAR(200) NOT NULL
  instructions TEXT
  total_marks SMALLINT NOT NULL
  duration_minutes SMALLINT NOT NULL
  question_ids UUID[]             -- ordered array of question IDs from bank
  is_published BOOLEAN DEFAULT false
  print_count SMALLINT DEFAULT 0
  created_by UUID FK
  created_at TIMESTAMPTZ now()
```

#### Staff Appraisal & CPD Tables
```
TABLE: staff_appraisals
  id, school_id, staff_id FK staff_profiles(id)
  academic_year_id FK, appraiser_id FK users(id)
  attendance_score NUMERIC(5,2)         -- 0-100
  academic_score NUMERIC(5,2)           -- avg class performance
  homework_score NUMERIC(5,2)           -- timely grading rate
  student_feedback_score NUMERIC(5,2)   -- anonymous survey avg
  peer_feedback_score NUMERIC(5,2)      -- peer review avg
  overall_score NUMERIC(5,2)            -- AI computed weighted avg
  grade VARCHAR(2)                      -- A+/A/B/C/D
  strengths TEXT                        -- AI-generated or manual
  improvement_areas TEXT
  training_recommendations TEXT[]
  increment_recommended BOOLEAN DEFAULT false
  increment_percent NUMERIC(5,2)
  status VARCHAR(20) DEFAULT draft CHECK IN (draft,in_review,finalized,shared_with_staff)
  staff_acknowledgement BOOLEAN DEFAULT false
  ai_generated BOOLEAN DEFAULT false
  created_at TIMESTAMPTZ now()

TABLE: cpd_records
  id, staff_id FK staff_profiles(id)
  training_name VARCHAR(300) NOT NULL
  training_type VARCHAR(30) CHECK IN (workshop,seminar,online,conference,certification)
  provider VARCHAR(200)
  start_date DATE NOT NULL, end_date DATE NOT NULL
  hours SMALLINT NOT NULL CHECK (1..500)
  certificate_url TEXT              -- Supabase Storage path
  verified_by UUID FK users(id)
  created_at TIMESTAMPTZ now()
```

#### OKR & Strategy Tables
```
TABLE: okrs
  id UUID PRIMARY KEY
  school_id UUID FK schools(id)         -- NULL = institution-level OKR
  institution_id UUID FK institutions(id)
  owner_id UUID FK users(id)
  title VARCHAR(300) NOT NULL           -- "Improve attendance to 90%"
  metric_name VARCHAR(100) NOT NULL     -- "Average Attendance %"
  baseline_value NUMERIC(10,2)
  target_value NUMERIC(10,2) NOT NULL
  current_value NUMERIC(10,2)           -- updated by AI daily
  progress_pct NUMERIC(5,2)             -- (current-baseline)/(target-baseline)*100
  start_date DATE NOT NULL
  target_date DATE NOT NULL
  status VARCHAR(20) DEFAULT active CHECK IN (active,achieved,missed,cancelled)
  ai_forecast_date DATE                 -- AI-predicted achievement date
  ai_recommendation TEXT                -- AI strategy suggestion
  data_source_table VARCHAR(80) NOT NULL -- e.g. attendance_summary
  data_source_query TEXT NOT NULL       -- SQL expression to compute current_value
  created_at TIMESTAMPTZ now()
  updated_at TIMESTAMPTZ now()
```

---

## PART C: POSTGRESQL STORED PROCEDURES & DATABASE FUNCTIONS

> All procedures: LANGUAGE plpgsql, SECURITY DEFINER where cross-table. Called via Supabase RPC.

### FUNCTION: get_school_health_score(p_school_id UUID) RETURNS JSONB
**Purpose:** Composite 0-100 health score across 5 weighted dimensions.
**Trigger:** Edge Function `nightly_health_score` (cron) + AI function call.

**Algorithm:**
```sql
-- 1. attendance_score (weight 30%)
SELECT AVG(percentage) FROM attendance_summary
WHERE school_id = p_school_id AND month = current_month AND academic_year_id = current;
-- Map 0-100% attendance → 0-100 score

-- 2. fee_score (weight 25%)
SELECT (SUM(paid_amount) / NULLIF(SUM(net_amount),0)) * 100
FROM student_fee_ledger WHERE school_id = p_school_id AND academic_year_id = current;

-- 3. academic_score (weight 25%)
SELECT AVG((total_obtained::float / NULLIF(max_marks,0)) * 100)
FROM student_marks sm JOIN exam_schedules es ON es.id = sm.exam_schedule_id
WHERE es.school_id = p_school_id AND es.academic_year_id = current;

-- 4. staff_score (weight 10%)
-- Avg: (staff_attendance_pct * 0.4) + (leave_utilization_pct * 0.3) + (appraisal_avg * 0.3)

-- 5. safety_score (weight 10%)
-- 100 - (open_critical_incidents * 10) - (overstay_visitors * 2) — clamped [0,100]
```

**Returns:**
```json
{
  "overall": 78.5,
  "grade": "B",
  "attendance": {"score": 85.2, "label": "Good"},
  "fee": {"score": 72.1, "label": "Average"},
  "academic": {"score": 80.0, "label": "Good"},
  "staff": {"score": 88.0, "label": "Excellent"},
  "safety": {"score": 90.0, "label": "Excellent"},
  "computed_at": "2026-03-23T07:15:00Z"
}
```

---

### FUNCTION: get_principal_dashboard_data(p_school_id UUID, p_date DATE) RETURNS JSONB
**Auth:** `auth.uid()` must have role=3 (Principal) AND school_id = p_school_id
**Purpose:** Single RPC for entire principal dashboard. Eliminates N+1 queries.

**Returns combined JSONB:**
```json
{
  "kpis": {
    "students_present_today": "INT — COUNT from student_attendance WHERE date=p_date AND status=present",
    "students_present_pct": "NUMERIC — students_present_today / total_enrolled",
    "fee_collected_today": "NUMERIC — SUM(amount) from fee_transactions WHERE DATE(payment_date)=p_date",
    "fee_collected_today_delta_pct": "NUMERIC — vs same day last week",
    "staff_on_leave": "INT — leave_applications WHERE date_range includes p_date AND status=approved",
    "active_visitors": "INT — visitors WHERE status=in AND school_id",
    "open_maintenance": "INT — maintenance_requests WHERE status!=resolved",
    "pending_leave_approvals": "INT",
    "pending_fee_waivers": "INT"
  },
  "class_attendance_grid": [
    {"class_name": "X", "section": "A", "attendance_by_date": [{"date": "...", "pct": 87.5}]}
  ],
  "fee_monthly_chart": [
    {"month": "Jan", "target": 500000, "collected": 430000, "collection_pct": 86}
  ],
  "staff_on_leave_today": [
    {"staff_id": "uuid", "name": "...", "dept": "...", "leave_type": "sick", "substitute_name": "..."}
  ],
  "upcoming_events": [
    {"title": "...", "type": "holiday|exam|event", "date": "...", "is_mandatory": true}
  ],
  "priority_alerts": [
    {"type": "dropout_risk|fee_overdue|attendance_dip", "severity": "high|medium", "message": "...", "student_id": null, "action_url": "..."}
  ],
  "health_score": "JSONB from get_school_health_score()"
}
```

---

### FUNCTION: mark_class_attendance(p_session_id UUID, p_records JSONB, p_teacher_id UUID) RETURNS JSONB
**Auth:** Verify teacher owns session (class_subject mapping), session status=open.

```sql
-- Authorization check
IF NOT EXISTS (
  SELECT 1 FROM attendance_sessions s
  JOIN class_subjects cs ON cs.class_id = s.class_id
  WHERE s.id = p_session_id AND cs.teacher_id = p_teacher_id AND s.status = 'open'
) THEN
  RAISE EXCEPTION 'access_denied: You are not authorized to mark this session';
END IF;

-- Upsert attendance records
INSERT INTO student_attendance (session_id, student_id, status, reason, marked_by)
SELECT p_session_id, (r->>'student_id')::UUID, r->>'status', r->>'reason', p_teacher_id
FROM jsonb_array_elements(p_records) r
ON CONFLICT (session_id, student_id) DO UPDATE
SET status = EXCLUDED.status, reason = EXCLUDED.reason, marked_by = p_teacher_id;

-- Update session totals, set status=submitted
UPDATE attendance_sessions SET
  total_present = (SELECT COUNT(*) FROM student_attendance WHERE session_id=p_session_id AND status='present'),
  total_absent  = (SELECT COUNT(*) FROM student_attendance WHERE session_id=p_session_id AND status='absent'),
  status = 'submitted', marked_at = now()
WHERE id = p_session_id;
```

**Returns:** `{success, session_id, total_present, absent_student_ids[]}` — absent IDs used by Edge Function to send parent alerts.

---

### FUNCTION: process_fee_payment(p_student_id UUID, p_amount NUMERIC, p_method VARCHAR, p_ref VARCHAR, p_collected_by UUID) RETURNS JSONB
**Auth:** user_role IN (3,4,5) AND has_permission(school.fee.collect). Role 6 CANNOT call directly.

```sql
BEGIN;
-- 1. Lock ledger row for update
SELECT * FROM student_fee_ledger WHERE student_id=p_student_id FOR UPDATE;

-- 2. Allocate to earliest unpaid instalment first (FIFO)
remaining := p_amount;
FOR inst IN SELECT * FROM fee_instalments
            WHERE ledger_id=ledger.id AND status!='paid' ORDER BY due_date LOOP
  pay_this := LEAST(remaining, inst.amount - inst.paid_amount + inst.late_fee);
  UPDATE fee_instalments
  SET paid_amount = paid_amount + pay_this,
      status = CASE WHEN paid_amount+pay_this >= amount THEN 'paid' ELSE 'partial' END
  WHERE id = inst.id;
  remaining := remaining - pay_this;
  EXIT WHEN remaining <= 0;
END LOOP;

-- 3. Generate receipt number: school_code-year_code-XXXXXX
receipt_no := school_code || '-' || year_code || '-' || LPAD(nextval('receipt_seq_'||school_id)::text, 6, '0');

-- 4. Insert fee_transactions record
-- 5. Update student_fee_ledger totals and status
-- 6. Insert audit_log entry
COMMIT;
```

**Returns:** `{success: true, receipt_no, transaction_id, balance: new_balance}`

---

### FUNCTION: get_user_permissions(p_user_id UUID, p_resource VARCHAR) RETURNS VARCHAR[]
**Purpose:** Return effective permissions for user on resource (role default + overrides).
**Used by:** All RLS policies, API authorization middleware.

```sql
-- 1. Fetch default permissions for user.role from role_permissions table
-- 2. Check privileges table for active overrides (valid_till IS NULL OR valid_till > now())
-- 3. If override found: REPLACE defaults with override.actions
-- 4. Return effective actions array

-- Example calls:
SELECT get_user_permissions('abc-uuid', 'fee.structure');  --> {read, write}
SELECT get_user_permissions('xyz-uuid', 'exam.marks');     --> {read}
```

---

### FUNCTION: check_ai_request_authorization(p_user_id UUID, p_function_name VARCHAR, p_context JSONB) RETURNS JSONB
**Language:** plpgsql STABLE SECURITY DEFINER
**Purpose:** Central authorization gate for EVERY AI function call. Called BEFORE Gemini executes any function.

**Authorization Matrix:**
```
mark_attendance      → roles [3,4,5], permission: attendance.mark
collect_fee          → roles [3,4,5], permission: fee.collect          — NOT role 6
process_payroll      → roles [3],     permission: payroll.process       — NOT 4,5,6
approve_leave        → roles [3,4],   permission: hr.leave.approve      — NOT 5,6
generate_report_card → roles [3,4,5], permission: exam.report_card      — NOT 6 (read only)
grant_privilege      → roles [3],     permission: admin.privileges       — NEVER 4,5,6
compare_branches     → roles [1,2] ONLY  — L3 and below CANNOT see cross-school data
send_circular (all)  → roles [3,4,5]. scope limited to own school
get_school_summary   → roles [1,2,3,4]. L5=own class only. L6=own child only
process_bank_transfer → roles [3] ONLY with 2FA confirmation required
```

**Student/Parent (role 6) BLOCKED functions:**
- ANY write/delete/update EXCEPT: own fee payment, own homework submission
- CANNOT: mark_attendance, collect_fee, enter_marks, send_circular, approve_*, process_*
- ALLOWED: check_fee_status (own only), get_bus_location (own route), ask_doubt, generate_study_plan

**Cross-school protection:**
- roles [3,4,5,6] context.school_id MUST match users.school_id
- Any mismatch → RAISE EXCEPTION 'unauthorized_cross_school_access'

**Returns:** `{authorized: BOOL, reason: VARCHAR, allowed_scope: JSONB}`

---

### FUNCTION: get_teacher_dashboard_data(p_teacher_id UUID, p_date DATE) RETURNS JSONB
**Returns:** Today's timetable, class attendance status per period, pending homework to check, upcoming exams, student at-risk alerts for own classes.

### FUNCTION: get_director_dashboard_data(p_institution_id UUID, p_date DATE) RETURNS JSONB
**Auth:** role=2 AND institution_id matches.
**Returns:** All schools' health scores, revenue comparison, attendance trends, top/bottom performing branches, AI cross-school insights.

---

## PART D: SECURITY — ROW LEVEL SECURITY (RLS) POLICIES

```sql
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance_sessions ENABLE ROW LEVEL SECURITY;
-- ... (all tables)

-- Example: users table RLS
CREATE POLICY users_own_read ON users FOR SELECT
  USING (id = auth.uid()
    OR (school_id = (SELECT school_id FROM users WHERE id = auth.uid())
        AND (SELECT role FROM users WHERE id = auth.uid()) BETWEEN 3 AND 5)
    OR (SELECT role FROM users WHERE id = auth.uid()) IN (0, 1, 2));

-- Example: student_profiles school isolation
CREATE POLICY student_profiles_school ON student_profiles FOR ALL
  USING (school_id = (SELECT school_id FROM users WHERE id = auth.uid()));

-- Example: fee_transactions — students see own only
CREATE POLICY fee_txn_own ON fee_transactions FOR SELECT
  USING (
    student_id IN (SELECT id FROM student_profiles WHERE user_id = auth.uid())
    OR (SELECT role FROM users WHERE id = auth.uid()) BETWEEN 3 AND 5
      AND school_id = (SELECT school_id FROM users WHERE id = auth.uid())
  );

-- Directors see all schools in their institution only
CREATE POLICY director_cross_school ON schools FOR SELECT
  USING (
    institution_id = (SELECT institution_id FROM users WHERE id = auth.uid())
    OR (SELECT role FROM users WHERE id = auth.uid()) IN (0, 1)
  );
```

---

## PART E: REST API SPECIFICATION

### 4.1 Standard Request Headers (ALL requests)
```
Authorization: Bearer {supabase_jwt_token}
  -- JWT contains: user_id, role, school_id, institution_id, email, iat, exp
  -- Expiry: 1 hour (access token), 7 days (refresh token)

Content-Type: application/json
apikey: {supabase_anon_key}

X-EduVerse-School-ID: {school_uuid}
  -- Verified against JWT school_id claim — mismatch → 403 Forbidden

X-EduVerse-App-Version: 4.0.0
X-EduVerse-Platform: android|ios|web|windows|macos|linux
Accept-Language: en|hi|gu|ta|te

-- Write operations only:
X-EduVerse-Idempotency-Key: {uuid}   -- Prevents duplicate payments/submissions on retry
```

### 4.2 API Routes — Complete List

| Method | Route | Auth Level | Purpose | Body/Params |
|--------|-------|-----------|---------|------------|
| POST | /auth/v1/otp | Public | Send phone OTP | {phone: "+919876543210", channel: "sms"} |
| POST | /auth/v1/verify | Public | Verify OTP, get JWT | {phone, token, type: "sms"} |
| POST | /auth/v1/token?grant_type=password | Public | Email+password login | {email, password} |
| POST | /auth/v1/token?grant_type=refresh_token | JWT | Refresh access token | {refresh_token} |
| POST | /auth/v1/logout | JWT | Logout all sessions | {} |
| GET | /rest/v1/users?select=*&id=eq.{id} | JWT+RLS | Get user profile | Query params |
| PATCH | /rest/v1/users?id=eq.{id} | JWT+RLS+own | Update own profile | {full_name?, avatar_url?, locale?} |
| GET | /rest/v1/student_profiles?school_id=eq.{id}&select=*,users(*) | JWT+RLS L3-L5 | List students | ?class_id=, ?search=, ?limit=, ?offset= |
| POST | /rest/v1/student_profiles | JWT L3-L4 | Add student | Full student object |
| PATCH | /rest/v1/student_profiles?id=eq.{id} | JWT L3-L4 write priv | Update student | Partial student fields |
| GET | /rest/v1/attendance_sessions?class_id=eq.{id}&date=eq.{date} | JWT L3-L5 | Get session | |
| POST | /rest/v1/rpc/mark_class_attendance | JWT L5 class teacher | Mark batch attendance | {session_id, records:[{student_id,status,reason}]} |
| GET | /rest/v1/attendance_summary?student_id=eq.{id} | JWT RLS | Get attendance summary | ?academic_year_id=, ?month= |
| GET | /rest/v1/rpc/get_attendance_report | JWT L3-L5 | Full attendance report | {school_id, class_id?, from_date, to_date, format?} |
| GET | /rest/v1/student_fee_ledger?student_id=eq.{id} | JWT RLS L3-L6 own | Get fee ledger | |
| POST | /rest/v1/rpc/process_fee_payment | JWT L3-L5 fee.collect priv | Record payment | {student_id, amount, method, ref, collected_by} |
| GET | /rest/v1/fee_transactions?school_id=eq.{id} | JWT L3-L5 fee.view | List transactions | ?student_id=, ?date=, ?method=, ?status= |
| POST | /rest/v1/rpc/generate_receipt_number | JWT L3-L5 | Get next receipt no | {school_id} |
| POST | /functions/v1/upi_collect_request | JWT L6 own only | Initiate UPI collect | {student_id, amount, vpa, purpose} |
| POST | /functions/v1/upi_payment_webhook | HMAC signed only | Confirm UPI payment | Signed payload from UPI gateway |
| GET | /functions/v1/upi_payment_status/{txn_ref} | JWT | Poll payment status | |
| POST | /rest/v1/rpc/create_exam_schedule | JWT L3-L4 | Create exam | {name, type, classes[], start_date, end_date} |
| POST | /functions/v1/scan_omr | JWT L3-L5 exam.grade | AI OMR grading | {exam_subject_id, student_id, image_base64} |
| POST | /functions/v1/evaluate_written_answer | JWT L3-L5 exam.grade | AI written eval | {exam_subject_id, student_id, image_urls[]} |
| POST | /rest/v1/rpc/bulk_enter_marks | JWT L4-L5 own subject | Batch marks entry | {exam_subject_id, marks_data:[{student_id,marks}]} |
| POST | /functions/v1/ai_chat | JWT all roles | AI chat message | {message, screen_context, session_id, history[]} |
| POST | /functions/v1/ai_function_execute | JWT+role check | Execute AI function | {function_name, parameters, confirmed} |
| GET | /rest/v1/ai_action_logs?school_id=eq.{id} | JWT L3+ only | Get AI activity log | ?action_module=, ?from=, ?to=, ?status= |
| GET | /rest/v1/vehicle_tracking?vehicle_id=eq.{id}&order=timestamp.desc&limit=1 | JWT all | Live bus location | |
| GET | /functions/v1/get_eta | JWT all | OSRM ETA calculation | {bus_lat, bus_lng, stop_lat, stop_lng} |
| POST | /rest/v1/visitors | JWT L3-L5 visitor.write | Log new visitor | Full visitor object |
| PATCH | /rest/v1/visitors?id=eq.{id} | JWT L3-L5 guard | Check out visitor | {status: "out", out_time} |
| POST | /functions/v1/send_notification | JWT L3-L5 comms.send | Send notification | {user_ids[], title, body, channels[], data{}} |
| PATCH | /rest/v1/notifications?id=eq.{id} | JWT own only | Mark as read | {is_read: true} |
| POST | /functions/v1/generate_pdf | JWT + report permission | Generate PDF report | {report_type, params, format} |
| GET | /functions/v1/export_data | JWT + export permission | Export data CSV/Excel | {entity, filters, format, school_id} |
| POST | /rest/v1/rpc/get_principal_dashboard_data | JWT L3 own school | Full dashboard data | {school_id, date} |
| POST | /rest/v1/rpc/get_teacher_dashboard_data | JWT L5 own context | Teacher dashboard | {teacher_id, date} |
| POST | /rest/v1/rpc/get_director_dashboard_data | JWT L2 own institution | Director intelligence | {institution_id, date} |

### 4.3 API Error Responses — Standard Format
```json
{
  "error": {
    "code": "fee_already_paid | unauthorized | school_mismatch | validation_error",
    "message": "Human readable message in Accept-Language",
    "details": {},
    "timestamp": "2026-03-24T10:00:00Z",
    "request_id": "uuid"
  }
}
```

**HTTP Status Codes:**
- 200 OK — Success
- 201 Created — Record created
- 400 Bad Request — Validation error
- 401 Unauthorized — Missing/invalid JWT
- 403 Forbidden — Insufficient role/permission or cross-school attempt
- 404 Not Found — Record not found
- 409 Conflict — Duplicate idempotency key
- 422 Unprocessable — Business rule violation
- 500 Internal — Server error (Edge Function crash)

---

## PART F: UPI DIRECT INTEGRATION — Zero Per-Transaction Cost Strategy

**Why direct UPI:** Avoids Razorpay 1.5-2% + GST per transaction. Education category MDR = 0% under RBI guidelines.

### Setup (One-Time per School)
1. School registers as UPI merchant with their bank (free, via bank's merchant portal)
2. School gets: Merchant UPI VPA (e.g. eduverset.dpsagra@sbi) + Merchant ID
3. Store in `schools.payment_config` JSONB: `{upi_vpa, merchant_id, bank_name}`

### Payment Flow — Option A: UPI Autopay / Collect Request
```
Step 1: Parent opens fee screen → selects instalment → taps [Pay via UPI]
Step 2: Flutter calls POST /functions/v1/upi_collect_request
        Body: {student_id, amount, vpa: parent_upi_id, purpose: "Fee Q1 2025-26", school_merchant_vpa}
Step 3: Edge Function sends UPI Collect Request to school's bank API
        (NPCI UPI APIs via bank partnership — ICICI/SBI/HDFC integration)
Step 4: Parent receives UPI collect notification on their UPI app (GPay/PhonePe/Paytm)
Step 5: Parent approves in UPI app → bank sends webhook to Edge Function
Step 6: POST /functions/v1/upi_payment_webhook (HMAC-SHA256 signed)
        Edge Function verifies signature → calls process_fee_payment() RPC → updates ledger
Step 7: Receipt generated, WhatsApp/SMS sent to parent
Step 8: Flutter polls GET /functions/v1/upi_payment_status/{txn_ref} every 3s for 5 min
```

### Option B: UPI QR Code (Walk-in Payment)
```
Accountant taps [Generate QR] → Edge Function generates UPI QR with amount encoded
Parent scans with any UPI app → payment hits school VPA
Webhook confirms → payment marked in system
QR valid for 15 minutes, single-use
```

### Option C: Razorpay Fallback
```
Used when: UPI collect not supported by parent's bank, or payment >₹1L limit
Flutter initializes Razorpay SDK with key_id
On success callback: POST /functions/v1/razorpay_webhook
Razorpay charges: 1.5% + 18% GST → school absorbs or passes to parent as "gateway fee"
```

### BBPS Integration (Bharat Bill Payment System)
```
For recurring fee collection via BBPS billers
School registers as BBPS biller (via bank)
Parents can pay from any BBPS-enabled app (including bank apps)
Reconciliation: BBPS sends daily settlement file → Edge Function processes
```

---

## PART G: CRITICAL BACKEND RULES

1. ALL timestamps stored as TIMESTAMPTZ UTC
2. ALL financial amounts in INR paise (INTEGER) in calculations; convert to rupees for display
3. ALL Supabase queries MUST filter by school_id (RLS also enforces as backup)
4. Cross-school data: users.school_id MUST match request school_id — verified in stored procedure
5. Director (L2) portal shows ONLY their institution's schools — NEVER other institutions
6. Face recognition: ML Kit face detection + stored VECTOR embedding from student_profiles → pgvector similarity
7. School Health Score is computed by stored procedure get_school_health_score() — NEVER compute in Flutter
8. PDF generation happens in Edge Function — NOT in Flutter (too heavy for device)
9. Offline writes go to Isar PendingWriteQueue — SyncManager flushes on reconnect
10. EVERY write operation through AI MUST call ErpFunctionExecutor.execute() which calls check_ai_request_authorization() first
