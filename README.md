# S-CoreBox

A movie tracking web application running on Cloudflare.

**Live:** https://scorebox-live.scorebox-live.workers.dev

Every movie carries a rating, its genres, a link to IMDb, and a summary of what
its reviewers actually said. As of November 2026 the API serves 697 titles.

## How it works

    TMDb API  -->  n8n workflow  -->  Cloudflare Worker  -->  D1 (SQLite)
                       |                     |
                  merge on movie ID     /api/movies  (read)
                                        /api/update-movies (write, key-protected)

- **Storage** - Cloudflare D1 (SQLite) with two tables, `movie_reviews` and `users`.
- **API** - a single Cloudflare Worker. `GET /api/movies` returns paginated records
  (`{movies, page, limit, total}`); `POST /api/update-movies` accepts a batch of
  titles and upserts them.
- **Ingestion** - an n8n workflow pulls popular titles from the TMDb API, fetches
  each title's reviews, splits and merges both streams on movie ID, then posts the
  batch to the Worker.
- **Posters** - served through a Metahub image proxy. IMDb blocks hotlinking and
  cross-origin requests, so pointing at the source URL leaves every poster broken.

## Notes on this repository

It holds the current Worker plus the earlier prototypes this project passed
through (a Flask + MySQL version under `app.py` / `models.py`, and a PHP version
under `*.php`). `wrangler.toml` binds the deployed Worker to its D1 database.
Earlier prototypes used a local MySQL instance and are not part of the deployed
app.

## Deploy

    npm install -g wrangler
    wrangler d1 execute scorebox-live --file=database/schema.sql
    wrangler deploy
