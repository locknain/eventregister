---
name: event-registration-system
description: Build and customize a two-file event registration system (public multi-page landing page plus private real-time admin dashboard) with Supabase cloud sync. Use when the user asks to create, update, or extend an event registration website, multi-event landing page, attendee form, or real-time admin console.
---

# Event Registration System

Build and maintain a separated, single-repo event registration system consisting of a public recipient-facing multi-page landing app (`index.html`) and a completely separate private real-time admin console (`admin.html`), optionally backed by Supabase.

## When to Use

- The user asks to build, update, or customize an event registration system or event landing page.
- The user wants separate public attendee pages and a hidden/private admin dashboard in one repository.
- The user wants multi-event support via URL parameters (`?event=slug`) with real-time registration tracking.

## Core Architecture & Design Standards

### 1. Strict Separation of Public & Admin Interfaces
- Never include any Admin button, link, tab, or admin logic inside `index.html`. Recipients only see the public event experience.
- Keep `admin.html` as a standalone file accessed directly by URL (e.g., `/admin.html`), protected by a login gate (PIN/password) and Supabase Row-Level Security (RLS).

### 2. Visual & UX Standards
- **Theme**: Automatic device light/dark mode (`prefers-color-scheme: dark`) using Matte White (`#F5F5F7` canvas, `#FFFFFF` cards) and Matte Black (`#09090B` canvas, `#111114` cards).
- **Typography**: `Inter` for UI/headings and `JetBrains Mono` for metrics, timestamps, and codes.
- **Navigation (`index.html`)**: Floating Apple Liquid Glass bottom pill navigation bar lifted safely above mobile gesture indicators (`env(safe-area-inset-bottom)`), plus mobile anti-zoom rules (`font-size: 16px` on inputs under 768px).
- **Clean Blank Slate (`admin.html`)**: Always initialize `admin.html` with zero pre-filled sample events or sample registrations (`0` KPIs) unless explicitly requested otherwise.

### 3. Public Event Landing Page (`index.html` - 3 Pages)
- Support dynamic event switching via `?event=<slug>` URL query parameters.
- **Page 1 (Register)**: Registration form collecting **Name**, **Position**, **Company**, and **Email**, syncing submissions in real time to the admin dashboard.
- **Page 2 (Itinerary)**: Event schedule timeline plus downloadable/previewable collaterals (such as presentation slides).
- **Page 3 (Company)**: Organizer details including **Company Name**, **Email Address**, and **HQ Address**.

### 4. Private Admin Console (`admin.html`)
- **Login Gate**: Require authentication before displaying any registration data.
- **Header Controls**: Include a prominent top-right **Supabase** connection button opening the Supabase Cloud Sync modal (with `Project URL`, `Anon / Publishable Key`, and one-click `Copy Table SQL`).
- **Real-Time Feed**: Display Page 1 registrant details (`Name`, `Email`, `Position`, `Company`, `Event`, `Time`) with live Supabase Realtime / `BroadcastChannel` updates, event filter dropdown, search, check-in toggle, CSV export, `+ New Event` creator, and `Copy Public Event Link` (which embeds `?event=<slug>` and optional Supabase params).

## Supabase Setup Schema

Provide this SQL via the `Copy Table SQL` action in `admin.html`:

```sql
grant usage on schema public to anon, authenticated;

create table if not exists public.event_registrations (
  id text primary key,
  event_slug text not null,
  name text not null,
  position text not null,
  company text not null,
  email text not null,
  checked_in boolean not null default false,
  created_at timestamptz not null default now()
);

grant all on table public.event_registrations to anon, authenticated;
alter table public.event_registrations enable row level security;

drop policy if exists "Public can insert registrations" on public.event_registrations;
create policy "Public can insert registrations"
  on public.event_registrations for insert
  to anon, authenticated
  with check (true);

drop policy if exists "Admin full read/update/delete" on public.event_registrations;
create policy "Admin full read/update/delete"
  on public.event_registrations for all
  to anon, authenticated
  using (true)
  with check (true);

notify pgrst, 'reload schema';
