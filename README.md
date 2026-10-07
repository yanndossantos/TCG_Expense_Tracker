# yanndossantos.github.io

## Supabase migration for multiple TCGs

The tracker stores the active game on expenses and budgets. Run this once in the Supabase SQL editor before deploying the updated page:

```sql
alter table public.expenses add column if not exists game text not null default 'riftbound';
alter table public.budgets add column if not exists game text not null default 'riftbound';

-- Only needed if the original table has this generated unique constraint.
alter table public.budgets drop constraint if exists budgets_user_id_month_key;

create unique index if not exists budgets_user_game_month_idx
	on public.budgets (user_id, game, month);

-- Required for editing an expense from the app.
alter table public.expenses enable row level security;
create policy "Users can update their own expenses"
on public.expenses for update
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

Existing records are automatically assigned to `riftbound`. The page currently offers `Riftbound` and `Other TCG`; add more options in `index.html` when needed.

## Keeping Supabase active

The repository includes a scheduled GitHub Actions workflow that queries Supabase once
per day to prevent the free project database from becoming inactive:
[`.github/workflows/keep-supabase-awake.yml`](.github/workflows/keep-supabase-awake.yml).

Add these repository secrets in **Settings > Secrets and variables > Actions**:

- `SUPABASE_URL`: the project URL, for example `https://your-project.supabase.co`
- `SUPABASE_ANON_KEY`: the project's publishable/anonymous key

The workflow can also be run manually from the **Actions** tab. It intentionally uses
the anonymous key and a read-only request; Row Level Security may return no rows, but
the request still validates that the API and database are reachable.