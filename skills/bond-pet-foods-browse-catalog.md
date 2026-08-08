---
name: bond-pet-foods-browse-catalog
description: >-
  Read the Bond Pet Foods online shop catalog anonymously — list products, filter and paginate
  them, resolve a single product by id or slug, and read the category, tag, brand and attribute
  taxonomies. No credentials required.
generated: '2026-08-08'
method: generated
source: openapi/bond-pet-foods-store-openapi.json
api: bond-pet-foods:store
base_url: https://www.bondpets.com/wp-json/wc/store/v1
auth: none
operations:
- getWcStoreV1Products
- getWcStoreV1ProductsById
- getWcStoreV1ProductsBySlug
- getWcStoreV1ProductsCategories
- getWcStoreV1ProductsCategoriesById
- getWcStoreV1ProductsTags
- getWcStoreV1ProductsBrands
- getWcStoreV1ProductsAttributes
- getWcStoreV1ProductsAttributesById
- getWcStoreV1ProductsAttributesByAttributeIdTerms
- getWcStoreV1ProductsCollectionData
- getWcStoreV1ProductsReviews
---

# Browse the Bond Pet Foods catalog

Every operation in this skill is a **read**, works **anonymously**, and was verified against the
live host on 2026-08-08.

## Before you start — what this catalog actually contains

This is the Bond Pet Foods **merchandise** store: branded apparel, a tote, and a coffee donation
item. It does **not** contain the fermented chicken protein Bond Pet Foods manufactures — that is
a B2B ingredient and is not represented on any API. At time of probe the catalog held 5 products,
one of which is named `Test Product` and is a live purchasable listing. Do not treat this catalog
as a product line description.

## 1. List products

`getWcStoreV1Products` — `GET /wc/store/v1/products`

Useful query parameters (all declared in the spec): `page`, `per_page` (max 100), `search`,
`slug`, `include`, `exclude`, `order`, `orderby`, `category`, `tag`, `brand`, `attributes`,
`min_price`, `max_price`, `stock_status`, `on_sale`, `rating`.

Read pagination from the **response headers**, not the body:

- `X-WP-Total` — total items
- `X-WP-TotalPages` — total pages at the requested `per_page`
- `Link` — RFC 8288 header with `rel="next"` / `rel="prev"`

Follow `Link: rel="next"` until it is absent. Do not guess page counts.

## 2. Resolve one product

Two operations, and they collide — read this carefully:

- `getWcStoreV1ProductsById` — `GET /wc/store/v1/products/{id}` (integer)
- `getWcStoreV1ProductsBySlug` — `GET /wc/store/v1/products/{slug}` (string)

They share one URL template and are disambiguated only by whether the segment is numeric. Pass an
integer for id lookup and a non-numeric slug for slug lookup.

A missing id returns **404** with:

```json
{"code":"woocommerce_rest_product_invalid_id","message":"Invalid product ID.","data":{"status":404}}
```

Branch on `code`, not on `message` — the message is localized.

## 3. Understand the product shape

The `Product` schema carries 36 fields. The ones that matter:

- `type` — `simple`, `variable` or `grouped`
- `variations[]` — child product ids, present only on `variable` products
- `parent` — set on a variation, pointing back at its parent
- `prices` — an object using the **smallest currency unit**, with `currency_minor_unit` telling
  you where the decimal goes. A `price` of `5500` with `currency_minor_unit: 2` is $55.00. Do not
  render `prices.price` directly.
- `is_purchasable`, `is_in_stock`, `has_options` — check all three before assuming a product can
  be bought
- `add_to_cart` — the parameters the cart routes expect for this product

## 4. Walk the taxonomies

`getWcStoreV1ProductsCategories`, `getWcStoreV1ProductsTags`, `getWcStoreV1ProductsBrands` and
`getWcStoreV1ProductsAttributes` list the term vocabularies. Categories are hierarchical —
`parent` is self-referential; walk it to build the tree.

Expect these to be **empty** on this store. At time of probe: one category (`Uncategorized`), zero
tags, zero brands, zero attributes, zero reviews. An empty array is a correct answer here, not an
error — do not retry it.

`getWcStoreV1ProductsCollectionData` returns aggregate facets (price range, term counts, rating
counts) for a filtered collection in one call. Prefer it over counting client-side.

## 5. Rules that apply to every call

- **No rate-limit signal exists.** No `RateLimit`, `X-RateLimit-*` or `Retry-After` header is ever
  returned, and no policy is published. Self-limit — a limit may exist and be enforced without
  warning.
- **No request id is returned.** There is nothing to cite in a support ticket, and there is no
  developer support channel regardless.
- **No caching.** Responses carry `cache-control: no-store, no-cache, must-revalidate` and no
  `ETag`. Conditional requests are not supported.
- **Errors are not RFC 9457.** The envelope is `{"code", "message", "data": {"status"}}` with media
  type `application/json`. See `errors/bond-pet-foods-problem-types.yml`.
- **CORS is open.** `access-control-allow-origin: *`, so a browser client can call this directly.
