---
name: bond-pet-foods-read-content
description: >-
  Read Bond Pet Foods company content anonymously through the WordPress REST API — news and press
  posts, site pages, media, taxonomies and site search — and join a product's editorial record to
  its commerce record. No credentials required for reads.
generated: '2026-08-08'
method: generated
source: openapi/bond-pet-foods-content-openapi.json
api: bond-pet-foods:content
base_url: https://www.bondpets.com/wp-json/wp/v2
auth: none
operations:
- getWpV2Posts
- getWpV2PostsById
- getWpV2Pages
- getWpV2PagesById
- getWpV2Media
- getWpV2MediaById
- getWpV2Categories
- getWpV2Tags
- getWpV2Search
- getWpV2Product
- getWpV2Types
- getWpV2Taxonomies
---

# Read Bond Pet Foods content

All reads below work **anonymously** and were verified against the live host on 2026-08-08.

## 1. News and press

`getWpV2Posts` — `GET /wp/v2/posts`

This is the company's news feed, surfaced on the site at `https://www.bondpets.com/news/`.

**Important:** many entries are press *coverage*, not first-party writing, and their `link` field
points at the **original publisher** — prnewswire.com, symrise.com, agfundernews.com,
vegconomist.com, youtube.com — not at bondpets.com. If you are collecting Bond Pet Foods'
own writing, check the host of `link` before attributing the piece to them.

Use `_fields` to keep responses small:

```
GET /wp/v2/posts?per_page=20&_fields=id,date,link,title,excerpt
```

Use `_embed` to inline the author, featured media and terms in a single request instead of
walking each edge:

```
GET /wp/v2/posts?per_page=20&_embed
```

`_fields`, `_embed` and `context` are WordPress core conventions. They are **not** declared in the
route argument schemas the host publishes, so they do not appear as parameters in the OpenAPI —
but all three were verified to work here.

## 2. Pages

`getWpV2Pages` — `GET /wp/v2/pages`

15 published pages, returned with fully rendered HTML in `content.rendered`. The substantive ones
are `our-mission`, `our-team`, `press`, `careers`, `faqs`, `contact` and `shop`. `parent` is
self-referential; walk it for hierarchy.

Content fields come back as **rendered HTML**, not plain text or markdown. Strip tags and
unescape HTML entities before using the text.

## 3. Search

`getWpV2Search` — `GET /wp/v2/search?search=<term>`

Covers **posts, pages and terms only**. It does **not** cover Store API products — search those
separately with the `search` parameter on `getWcStoreV1Products`. There is no unified
cross-surface search on this host.

## 4. Media

`getWpV2Media` — `GET /wp/v2/media`

`source_url` is the direct asset URL; `media_details.sizes` carries the generated renditions.
`post` points back at the Post or Page the asset was uploaded to.

## 5. Join editorial to commerce

`getWpV2Product` — `GET /wp/v2/product`

This is the **same** WooCommerce product record as the Store API's, projected as a WordPress post
type. Neither projection is a superset:

| Field group | `wp/v2/product` | `wc/store/v1/products` |
|---|---|---|
| `content`, `excerpt`, `featured_media`, `template` | yes | no |
| `prices`, `sku`, `stock`, `attributes`, `variations`, `add_to_cart` | no | yes |

They share one integer id space. To get both, call both and **join on `id`**. See
`data-model/bond-pet-foods-data-model.yml` → `cross_surface`.

## 6. What you cannot read

These return **401** anonymously and there is no way to obtain credentials — Bond Pet Foods runs
no developer program and issues no API keys:

- `GET /wp/v2/users` → `rest_user_cannot_view`
- `GET /wp/v2/settings` → `rest_forbidden`
- `GET /wp-abilities/v1/abilities` → `rest_forbidden`

Because `/wp/v2/users` is locked down, the `author` field on a post is a bare integer you cannot
resolve. Use `_embed` — embedded author data is returned where the site permits it — or accept the
id as opaque.

## 7. Rules that apply to every call

- **Writes exist in the spec but are not available to you.** Every `POST`/`PUT`/`PATCH`/`DELETE`
  route in this API requires a cookie nonce or an Application Password. Treat this API as
  read-only.
- **No rate-limit signal, no request id, no ETag.** See
  `conventions/bond-pet-foods-conventions.yml`.
- **Errors are `{"code","message","data":{"status"}}`**, not RFC 9457. Branch on `code`.
- **Pagination is `page`/`per_page` with `X-WP-Total`, `X-WP-TotalPages` and a `Link` header.**
