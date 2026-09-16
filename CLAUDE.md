# CLAUDE.md — Saracen Stockist Finder

Guidance for working in this repo. This folder is the data feed for the Saracen
Horse Feeds Stockist Finder. Every stockist change is made in **two** places and
kept in sync: the GeoJSON file here, and the Webflow CMS.

## Webflow: default site and workspace

- **Workspace ID:** `6332cf5e4bdcdf01092d999b`
  (the only workspace this connection can see; contains ~317 sites)
- **Default site (the edit target):** Product Finder Testing
  - short name: `saracen-product-finder`
  - site ID: `69dcb28ab6ec8cf13fc33e3f`
  - **VERIFY:** the display name contains "Testing" and the site has no custom
    domain. Confirm this is the live Stockist Finder site before publishing. If
    the live site is a different one (for example in a separate "Clients"
    workspace), update this file with the correct site ID.
- **Stockist collection:** Distributor Locations, ID `69dcb28ab6ec8cf13fc33fcb`
- **Related collection:** Distributor Regions, ID `69dcb28ab6ec8cf13fc3400f`

## Off-limits sites (deny by default)

- Only the **default site** above may be created, updated or published against.
- **Every other site in the workspace is off-limits** unless the user explicitly
  authorises it in the current session. Do not create, update, publish or delete
  anything on any other site.
- Explicitly do **not** touch: **`saracenhorsefeeds`** (production marketing
  site), site ID `684c0274e4d52b0d2fe5890f`. It is the nearest neighbour and the
  easiest to edit by mistake.

## Field mapping (GeoJSON <-> Distributor Locations)

| GeoJSON | Webflow field slug | Notes |
| --- | --- | --- |
| `properties.name` | `name` | required |
| `id` | `slug` | kebab-case, required |
| `properties.postcode` | `post-code` | |
| `properties.phone` | `phone-number` | |
| `properties.website` | `website` | Link field |
| `properties.area` | `area` | Option field, set by option ID (see below) |
| `geometry.coordinates[1]` | `latitude` | latitude is the second value |
| `geometry.coordinates[0]` | `longitude` | longitude is the first value |
| (none) | `prefered-email` | Webflow only, usually empty |

### Area option IDs

| Area | Option ID |
| --- | --- |
| Scotland | `00f6780b32ee38a98051fca94f64c508` |
| South West | `df9fd890d88b59e535d077b09159e031` |
| East Anglia | `e499f098df7c535d9673114421b0e52c` |
| Eastern Region | `eb30b5506e790a1f68e96235828d6d62` |
| South East | `8852f7d6fd763cf029900b1a33fe673f` |
| Southern | `c8dcd1478444329fedecbd5e7289729a` |
| Northern Region | `4efcfa969ac5bcc37da3d002fc658c7b` |
| Midlands & North Wales | `28e4ed8de6097eb3952ac3b331083d7a` |
| South Wales | `5d0ae2bf160505d4e2465c794b1eb0b4` |

## Update workflow (do both sides)

1. **GeoJSON:** add, update or remove the `Feature` (geocode the postcode into
   `[longitude, latitude]`), then commit and push to the working branch.
2. **Webflow (Distributor Locations):**
   - Add: create the item, then publish it.
   - Update: find the item by slug, update it, then publish.
   - Delete: delete the item.

## Gotchas

- **Coordinate order:** GeoJSON stores `[longitude, latitude]`; Webflow has
  separate `latitude` and `longitude` numbers. Do not swap them.
- **`orderInOnly` flag** exists in the GeoJSON only. The Distributor Locations
  collection has no matching field, so that badge lives only in the GeoJSON
  unless a field is added to the collection.
- The two sources can drift (GeoJSON and CMS item counts have differed in the
  past). When in doubt, cross-check a recent change against the live CMS.

## MCP connection notes

- In a local Claude Code setup, add the server with:
  `claude mcp add --transport http webflow https://mcp.webflow.com/mcp`
  then authorise via `/mcp` and pick the correct workspace on the consent screen.
- In the managed remote (web) environment the Webflow connector is injected by
  the platform, so `claude mcp list` shows nothing even though the tools work,
  and OAuth cannot be run from that session.
