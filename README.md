# recipes.

A minimal, read-only recipe site. It's a single static page that loads recipes live from a Supabase database.

**Live:** https://recipes.luciusschaerer.space

## How it works

- `index.html` is the whole app: HTML, CSS and plain JavaScript, with no build step.
- On page load, the browser fetches recipes straight from Supabase using [supabase-js](https://github.com/supabase/supabase-js) and the project's publishable key.
- New or edited recipes show up on the next page load. Nothing needs to be redeployed.
- Routing uses the URL hash: `#/r/<recipe-id>` opens a recipe, and anything else shows the overview.

## Database

Three tables in the `public` schema:

| Table                | Contents                                                       |
| -------------------- | -------------------------------------------------------------- |
| `recipes`            | title, description, notes, source URL, SVG icon, status, owner |
| `recipe_ingredients` | amount, unit, name, note, section, position                    |
| `recipe_steps`       | instruction, position                                          |

### Public access

Anonymous visitors get read-only access through row-level security:

- Only recipes with `status = 'processed'` that belong to the owner's account are visible.
- Ingredients and steps are visible only for those recipes.
- Visitors can't read private columns of `recipes`, such as `user_id`, `source_text` and `source_image_path`. Only the columns the site displays are granted to the `anon` role.
- Visitors can't insert, update or delete anything.

The key in `index.html` is the **publishable** key. It's meant to be public, and access is enforced by the policies above.

If you add a column to `recipes` that the site should show, grant it to visitors in the Supabase SQL editor (replace `new_column` with the column's name):

```sql
grant select (new_column) on public.recipes to anon;
```

## Run locally

```bash
python3 -m http.server 5173 --bind 127.0.0.1
```

Then open http://localhost:5173.

## Deploy

GitHub Pages serves the `main` branch from the repository root, on the custom domain set in `CNAME`. Every push to `main` goes live within about a minute.
