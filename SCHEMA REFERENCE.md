# Quratto · Schema Reference

**Status:** Authoritative — built from live database dump (Layers 1, 2 & 3 complete)
**Tables:** 11 · `admins`, `category_groups`, `categories`, `profiles`, `profile_preferences`, `venues`, `experiences`, `ingestion_sources`, `agent_runs`, `saved_experiences`, `member_schedules`
**Conventions:** all timestamps `timestamptz` · `citext` for slugs/emails · soft-delete via `is_active` · `updated_at` maintained by trigger

---

## 1 · category_groups
The 7 top-level experience groups. Editable from admin. Soft-delete via `is_active`.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| slug | citext | NOT NULL | — | **PK**. Immutable identifier. e.g. `cultural_experiences` |
| name | text | NOT NULL | — | Editable display name |
| description | text | null | — | |
| emoji | text | null | — | |
| icon | text | null | — | |
| hero_image_url | text | null | — | |
| sort_order | smallint | NOT NULL | 100 | Lower = first |
| is_active | boolean | NOT NULL | true | Soft-delete |
| deprecated_at | timestamptz | null | — | |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | trigger-maintained |

**The 7:** cultural_experiences · markets_seasonal_events · culinary_experiences · social_evenings · active_pursuits · coastal_living · wellness_restoration

---

## 2 · categories
~35 sub-categories nested under groups. Editable. Soft-delete via `is_active`.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| slug | citext | NOT NULL | — | Unique. Immutable. e.g. `golf` |
| name | text | NOT NULL | — | Editable display name |
| description | text | null | — | |
| examples | text[] | NOT NULL | `{}` | e.g. {Private courses, 18-hole rounds} |
| icon | text | null | — | |
| category_group_slug | citext | NOT NULL | — | **FK** → category_groups.slug (cascade on update) |
| sort_order | smallint | NOT NULL | 100 | |
| is_active | boolean | NOT NULL | true | |
| deprecated_at | timestamptz | null | — | |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | |

---

## 3 · profiles
Member identity only — all preferences live in `profile_preferences`.
*Note: large gaps in column positions (22→90→98) reflect dropped preference columns from normalization. Healthy.*

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | — | **PK**, **FK** → auth.users.id |
| first_name | text | null | — | |
| last_name | text | null | — | |
| display_name | text | null | — | |
| pronouns | text | null | — | |
| date_of_birth | date | null | — | |
| age_range | text | null | — | |
| gender | text | null | — | |
| mobile | text | null | — | |
| mobile_verified | boolean | null | false | |
| whatsapp_opt_in | boolean | null | false | |
| city | text | null | 'Miami' | |
| neighborhood | text | null | — | |
| timezone | text | null | 'America/New_York' | |
| locale | text | null | 'en' | |
| language | text | null | 'en' | |
| home_base | text | null | — | |
| membership_status | text | null | 'pending' | pending/active/paused/cancelled |
| joined_at | timestamptz | null | — | |
| referral_by | text | null | 'Quratto Admin' | free-text name |
| relationship_status | text | null | — | |
| contact_channel | text | null | 'email' | |
| notification_frequency | text | null | 'weekly' | global default; per-category overrides in prefs |
| newsletter_opt_in | boolean | null | true | |
| marketing_opt_in | boolean | null | false | |
| concierge_opt_in | boolean | null | true | |
| curation_notes | text | null | — | curator-written, internal |
| member_bio | text | null | — | |
| last_active_at | timestamptz | null | — | |
| created_at | timestamptz | null | now() | |
| updated_at | timestamptz | null | now() | |
| preferences_updated_at | timestamptz | null | — | touched by prefs trigger |
| email | text | null | — | mirrors auth.users.email |
| email_verified | boolean | null | false | |

---

## 4 · profile_preferences
One row per member × sub-category (max ~35/member). Member-owned (RLS).

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| profile_id | uuid | NOT NULL | — | **FK** → profiles.id (cascade delete) |
| category_id | uuid | NOT NULL | — | **FK** → categories.id |
| interest_level | smallint | NOT NULL | 2 | 0=mute 1=curious 2=interested 3=love |
| frequency | text | null | — | rarely/monthly/weekly/often |
| skill_level | text | null | — | beginner/intermediate/advanced |
| group_size | text | null | — | solo/couple/small_group/any |
| budget_tier | text | null | — | accessible/elevated/exclusive |
| time_of_day | text | null | — | morning/afternoon/evening/any |
| notify_today | boolean | NOT NULL | false | |
| notify_weekly | boolean | NOT NULL | true | |
| notify_monthly | boolean | NOT NULL | false | |
| notes | text | null | — | member's own words |
| preferred_days | text[] | null | `{}` | {mon,wed,sat,sun} |
| accessibility_notes | text | null | — | |
| is_active | boolean | NOT NULL | true | pause without losing settings |
| sort_order | smallint | NOT NULL | 100 | member-controlled order |
| last_suggested_at | timestamptz | null | — | anti over-notification |
| last_engaged_at | timestamptz | null | — | behavioral signal |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | |

**Unique:** (profile_id, category_id)

---

## 5 · venues
Slim anchor table — URL reference + dedup, not full transcription. Public read.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| name | text | NOT NULL | — | |
| slug | citext | NOT NULL | — | Unique |
| url | text | null | — | canonical website |
| booking_url | text | null | — | |
| google_maps_url | text | null | — | |
| instagram_url | text | null | — | |
| phone | text | null | — | |
| email | text | null | — | |
| city | text | NOT NULL | 'Miami' | |
| neighborhood | text | null | — | |
| address | text | null | — | plain text, referenced not transcribed |
| lat | numeric(9,6) | null | — | |
| lng | numeric(9,6) | null | — | |
| category_id | uuid | null | — | **FK** → categories.id |
| curated_score | numeric(2,1) | null | — | 1.0–5.0 internal quality |
| curator_notes | text | null | — | internal |
| contact_name | text | null | — | |
| is_active | boolean | NOT NULL | true | |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | |

---

## 6 · experiences
The core curated unit. Agents draft → curators approve → members discover.
*Column position gaps (1→3, 8→10, 28→30) = dropped legacy columns: section, approved, event_url, starts_on, location, maps_url. Healthy.*

### Core
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| title | text | NOT NULL | — | |
| description | text | NOT NULL | — | |
| url | text | null | — | direct reference URL |
| submitted_by | uuid | null | — | **FK** → auth.users.id |
| slug | citext | null | — | SEO segment |
| category_id | uuid | null | — | **FK** → categories.id |
| venue_id | uuid | null | — | **FK** → venues.id (null = venue-less) |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | |

### Scheduling
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| starts_at | timestamptz | null | — | |
| ends_at | timestamptz | null | — | |
| is_recurring | boolean | NOT NULL | false | |
| recurrence_rule | text | null | — | human-readable |
| featured_on | date | null | — | Today in Miami date |

### Curation signals
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| price_tier | text | null | — | complimentary/accessible/elevated/exclusive |
| audience | text | null | — | couples/solo/small_group/families_mature/all |
| quratto_score | numeric(2,1) | null | — | 1.0–5.0 |
| curator_notes | text | null | — | internal |
| status | text | NOT NULL | 'draft' | draft/in_review/approved/featured/archived/rejected |
| city | text | NOT NULL | 'Miami' | |

### Media
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| hero_image_url | text | null | — | |
| video_url | text | null | — | |

### Location & contact (venue-less)
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| location_text | text | null | — | |
| contact_email | text | null | — | |
| contact_phone | text | null | — | |
| contact_name | text | null | — | |
| booking_url | text | null | — | |
| google_maps_url | text | null | — | |
| instagram_url | text | null | — | |

### Agent block (provenance / confidence / dedup / freshness)
| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| source_url | text | null | — | exact page scraped |
| source_name | text | null | — | Eventbrite, venue, publication |
| discovered_by | text | null | — | agent id |
| discovered_at | timestamptz | null | — | |
| agent_confidence | numeric(3,2) | null | — | 0.00–1.00 |
| extraction_quality | text | null | — | complete/partial/needs_review |
| external_id | text | null | — | source's own ID — dedup |
| dedup_hash | text | null | — | hash(title+date+venue) — dedup |
| last_seen_at | timestamptz | null | — | freshness |
| source_published_at | timestamptz | null | — | |
| expires_at | timestamptz | null | — | auto-archive cutoff |
| raw_payload | jsonb | null | — | raw extraction |

**Unique (dedup):** (source_name, external_id) where not null · (dedup_hash) where not null

---

## 7 · admins
Access control. Presence = admin.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| user_id | uuid | NOT NULL | — | **PK**, **FK** → auth.users.id |
| added_at | timestamptz | NOT NULL | now() | |

---

## 8 · ingestion_sources
**(Layer 2)** Configuration for where autonomous agents pull from. Editable, pausable. Admin-only.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| name | text | NOT NULL | — | e.g. Eventbrite Miami, PAMM Calendar |
| kind | text | NOT NULL | — | rss / api / scrape / manual |
| url | text | null | — | feed / endpoint / page |
| default_category_id | uuid | null | — | **FK** → categories.id (SET NULL) — category hint |
| is_active | boolean | NOT NULL | true | soft-pause |
| last_run_at | timestamptz | null | — | last processed by an agent |
| notes | text | null | — | |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | trigger-maintained |

---

## 9 · agent_runs
**(Layer 2)** Log of each agent execution. One row per source per run. Admin read; agents write via service_role.

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| source_id | uuid | null | — | **FK** → ingestion_sources.id (SET NULL) |
| agent_name | text | NOT NULL | — | e.g. agent-scraper-v1 |
| status | text | NOT NULL | 'running' | running / succeeded / failed |
| started_at | timestamptz | NOT NULL | now() | |
| finished_at | timestamptz | null | — | null while running |
| items_created | integer | NOT NULL | 0 | new experiences written |
| items_updated | integer | NOT NULL | 0 | existing refreshed |
| error_text | text | null | — | failure reason if status=failed |
| created_at | timestamptz | NOT NULL | now() | |

**Human curation trail:** reuses `experiences.curator_notes` (no separate audit table — by design).

**Pipeline flow:**
```
ingestion_sources (config) → agent_runs (execution) → experiences (status=in_review)
  → curator reviews queue [select * from experiences where status='in_review' order by agent_confidence desc]
  → approve/reject, decision appended to experiences.curator_notes
```

---

## 10 · saved_experiences
**(Layer 3)** Member bookmarks. One row per (member, experience). Member-owned (RLS).

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| profile_id | uuid | NOT NULL | — | **FK** → profiles.id (CASCADE) |
| experience_id | uuid | NOT NULL | — | **FK** → experiences.id (CASCADE) |
| created_at | timestamptz | NOT NULL | now() | |

**Unique:** (profile_id, experience_id)

---

## 11 · member_schedules
**(Layer 3)** Member personal curated calendar. Holds Quratto + custom entries. Snapshot fields make rows export-ready as `.ics` events. Member-owned (RLS).

| Col | Type | Null | Default | Notes |
|---|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() | **PK** |
| profile_id | uuid | NOT NULL | — | **FK** → profiles.id (CASCADE) |
| experience_id | uuid | null | — | **FK** → experiences.id (SET NULL). Null = custom entry |
| title | text | NOT NULL | — | from experience, or member-typed |
| description | text | null | — | |
| location_text | text | null | — | |
| starts_at | timestamptz | NOT NULL | — | required for calendar export |
| ends_at | timestamptz | null | — | |
| all_day | boolean | NOT NULL | false | |
| status | text | NOT NULL | 'planned' | planned/attended/skipped/cancelled |
| notes | text | null | — | private member note |
| reminder_at | timestamptz | null | — | in-app nudge |
| created_at | timestamptz | NOT NULL | now() | |
| updated_at | timestamptz | NOT NULL | now() | trigger-maintained |

**Check:** experience_id IS NOT NULL OR title IS NOT NULL (must be a real experience or a titled custom entry)

**Export model:** snapshot fields (title/starts_at/ends_at/location_text) live on the row so an `.ics` export is a simple read — stable even if the source experience changes or is deleted. One-way export only; no sync-state tracking.

---

## Foreign keys (CONFIRMED)
| Child.column | Parent | On Update | On Delete |
|---|---|---|---|
| categories.category_group_slug | category_groups.slug | CASCADE | RESTRICT |
| profile_preferences.profile_id | profiles.id | NO ACTION | CASCADE |
| profile_preferences.category_id | categories.id | NO ACTION | RESTRICT |
| venues.category_id | categories.id | NO ACTION | SET NULL |
| experiences.category_id | categories.id | NO ACTION | SET NULL |
| experiences.venue_id | venues.id | NO ACTION | SET NULL |
| experiences.submitted_by | auth.users.id | — | — (events_submitted_by_fkey) |
| profiles.id | auth.users.id | — | — (profiles_id_fkey) |
| admins.user_id | auth.users.id | — | — (admins_user_id_fkey) |
| ingestion_sources.default_category_id | categories.id | NO ACTION | SET NULL |
| agent_runs.source_id | ingestion_sources.id | NO ACTION | SET NULL |
| saved_experiences.profile_id | profiles.id | NO ACTION | CASCADE |
| saved_experiences.experience_id | experiences.id | NO ACTION | CASCADE |
| member_schedules.profile_id | profiles.id | NO ACTION | CASCADE |
| member_schedules.experience_id | experiences.id | NO ACTION | SET NULL |

**Delete-rule logic:** member delete cascades their prefs · category delete blocked while referenced (forces soft-delete) · venue delete nulls the experience link, experience survives.

## Indexes (CONFIRMED)
**category_groups:** pkey(slug)
**categories:** pkey(id) · slug_key(unique) · group_idx(group_slug where active) · active_sort_idx(sort_order where active)
**profiles:** pkey(id) · city · last_active · membership · neighborhood
**profile_preferences:** pkey(id) · unique(profile_id,category_id) · profile_idx · category_idx · notify_today · notify_weekly · last_suggested(weekly) · last_suggested_monthly · budget_tier · group_size · preferred_days(gin) · active_profile · sort_order
**venues:** pkey(id) · slug_key(unique) · slug_idx · category_idx(where active) · city_neighborhood_idx(where active)
**experiences:** pkey(id, named `events_pkey`) · slug_key(unique) · slug_idx · category · city · status · starts_at · venue · featured_on · expires · review_queue · source · source_external_uniq · dedup_hash_uniq · fts(gin) · submitted_by(named `idx_events_submitted_by`)
**ingestion_sources:** pkey(id) · active_idx(kind where active)
**agent_runs:** pkey(id) · source_idx(source_id, started_at desc) · status_idx(status, started_at desc)
**saved_experiences:** pkey(id) · unique(profile_id, experience_id) · profile_idx(profile_id, created_at desc)
**member_schedules:** pkey(id) · profile_idx(profile_id, starts_at) · upcoming_idx(profile_id, starts_at where planned) · experience_idx(where experience_id not null)

### Optional cosmetic cleanups (non-urgent)
```sql
alter index events_pkey rename to experiences_pkey;
alter index idx_events_submitted_by rename to experiences_submitted_by_idx;
drop index if exists experiences_slug_idx;  -- redundant with experiences_slug_key
```

## RLS (confirmed)
| Table | Policies |
|---|---|
| admins | read |
| category_groups | public read |
| categories | public read |
| venues | public read |
| profiles | read own · update own · service role all |
| profile_preferences | read/insert/update/delete own |
| experiences | public read · member insert · member update own draft · admin all |
| ingestion_sources | admin read · admin all (agents write via service_role) |
| agent_runs | admin read (agents write via service_role) |
| saved_experiences | read/insert/delete own |
| member_schedules | read/insert/update/delete own |

## Open items
1. **Profile-creation trigger** on auth.users signup — VERIFY it exists (no user INSERT policy on profiles by design). Run:
   `select tgname from pg_trigger where tgrelid = 'auth.users'::regclass and not tgisinternal;`
2. FKs + indexes — ✅ CONFIRMED against live DB.
3. Optional: cosmetic index renames (see Indexes section).
