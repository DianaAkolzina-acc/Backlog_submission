# GBS AI Request Portal

A GitHub Pages app for submitting and prioritising AI use case requests. Tickets are AI-reviewed by a local Ollama model before being saved to Supabase. Admins can view, rank, and edit tickets on a drag-and-drop prioritisation board.

## Live site

`https://dianakolzina-acc.github.io/Backlog_submission/`

Enable GitHub Pages: **repo Settings → Pages → Source: Deploy from branch → branch: `main`, folder: `/ (root)`**

---

## Setup — do this once

### 1. Supabase (ticket storage)

1. Create a free project at [supabase.com](https://supabase.com)
2. Go to **Settings → API** and copy your **Project URL** and **anon public key**
3. Open `index.html` and fill in the config at the top:
   ```js
   const SUPABASE_URL  = "https://YOUR_PROJECT.supabase.co";
   const SUPABASE_ANON = "YOUR_ANON_KEY";
   ```
4. In the Supabase **SQL editor**, run:
   ```sql
   create table tickets (
     id               bigint generated always as identity primary key,
     name             text,
     description      text,
     priority         int,
     access           text,
     "dataSources"    text,
     "painPoints"     text,
     workstream       text,
     "pointsOfContact" text,
     status           text default 'pending_review',
     rank             int,
     effort           text,
     created_at       timestamptz default now()
   );
   alter table tickets enable row level security;
   create policy "public read"   on tickets for select using (true);
   create policy "public insert" on tickets for insert with check (true);
   create policy "public update" on tickets for update using (true);
   ```

### 2. Ollama (local AI reviewer)

Each person who submits a ticket needs Ollama running on their machine for the AI review step. If it's not running, the ticket is saved as **"Pending review"** for manual admin review instead.

1. Install Ollama from [ollama.com](https://ollama.com)
2. Pull the model: `ollama pull llama3.2`
3. Start with CORS allowed from GitHub Pages:
   - **Windows:** `set OLLAMA_ORIGINS=* && ollama serve`
   - **Mac/Linux:** `OLLAMA_ORIGINS=* ollama serve`

### 3. Admins

Edit the `ADMINS` array in `index.html` to add/remove admin users and passwords.

---

## How it works

| Step | What happens |
|------|-------------|
| User submits form | Ollama reviews the ticket for completeness |
| Ollama needs more info | User answers clarifying questions (conversation history preserved) |
| Ollama approves | Ticket saved to Supabase with status `approved` |
| Ollama offline | Ticket saved as `pending_review` for admin to assess |
| Admin logs in | Sees all tickets, can filter by status/workstream |
| Admin ranks | Drag-and-drop or type rank number; saves instantly to Supabase |
| Admin edits | Can update any field including status and effort estimate |
