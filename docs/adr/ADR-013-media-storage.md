# ADR-013 · Media storage: images on Cloudflare R2 (+ Cloudflare Images), videos as external links

| | |
|---|---|
| **Status** | ✅ Accepted (updated Nov 2026; decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-13 in `docs/DECISIONS.md` |
| **Supersedes** | Earlier decision to use Supabase Storage (it had no separate ADR) |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Storage and delivery of images and videos |
| **Risk tier** | High (client and coach uploads, possibly progress photos) |

## Context

The app stores exercise thumbnails, coach and client uploads in calendar threads and profile photos, and shows exercise demo videos. Image traffic from coaches and clients can grow the bill, and egress was identified as the main Supabase cost risk (ADR-017).

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review. Differences between options A, B and C are small, within the margin of scoring error.

| Criterion (weight) | A. R2 + Cloudflare Images transformations | B. Supabase Storage | C. Google Cloud Storage + CDN | D. Images stored in Cloudflare Images |
|---|---|---|---|---|
| Security / privacy gate | Pass (private bucket, signed URLs) | Pass | Pass | Pass |
| Security (20%) | 4 | 4 | 4 | 4 |
| Maintainability (20%) | 3 | 5 | 4 | 4 |
| Reliability (15%) | 4 | 4 | 4 | 4 |
| Performance (15%) | 5 | 3 | 4 | 5 |
| Scalability (10%) | 5 | 3 | 5 | 4 |
| Cost (10%) | 5 | 3 | 3 | 2 |
| Integration (10%) | 3 | 5 | 4 | 3 |
| **Weighted score** | **4.05** | **3.95** | **4.00** | **3.85** |

## Decision

- **Images** (exercise thumbnails, uploads in calendar threads, profile photos) are stored on **Cloudflare R2**.
- **Cloudflare Images** transformations resize and convert them on the fly (thumbnail sizes, WebP/AVIF), reading from R2.
- **Exercise demo videos** stay external links (YouTube unlisted or Vimeo).
- **Supabase** stores only the file key and metadata (owner, `gym_id`, client, activity, size, type), not the file.

**Uploads:** the web app asks the API for an upload URL; the API checks the user's gym and permission and returns a short-lived presigned R2 URL; the browser uploads straight to R2; the web app tells the API it is done; the API saves the key and metadata.

| Bucket | Contents | Access |
|---|---|---|
| `fitsphere-public` | Exercise library thumbnails, prebuilt program images | Public, via a custom domain with Cloudflare cache |
| `fitsphere-private` | Client and coach uploads | Private; API returns short-lived signed URLs after a permission check |

Object keys are prefixed `gyms/{gym_id}/clients/{client_id}/…`.

## Why

- R2 has no egress fees, so serving images does not grow the bill with traffic, and Supabase egress stays for database traffic.
- Free tiers (per the decision log, to be re-verified): 10 GB storage, 1M writes and 10M reads a month on R2; 5,000 unique transformations a month on Cloudflare Images.

## Rejected

- **B. Supabase Storage (previous decision):** small free quota, and image traffic would share egress with database traffic. Simplest to run if volumes stay tiny.
- **C. Google Cloud Storage + CDN:** same cloud as the API, but egress is charged.
- **D. Storing images in Cloudflare Images itself:** paid plan; R2 plus transformations is cheaper at our size.
- **Hosting videos on R2:** possible later; large files would use up the free storage and need streaming.

## Consequences

- New secrets in Google Secret Manager (R2 account ID, access key, secret key, bucket names); never in the web app.
- Keep the number of image widths small (for example three); each width counts as a unique transformation.
- R2 CORS allows uploads only from the web app's domain.
- Deleting a client or gym also deletes their R2 objects.
- Upload rules: validate type and size, scan, serve private files via signed URLs (see `docs/best-practices/01-coding-best-practices.md`).

## Related decisions

ADR-003, ADR-004, ADR-005, ADR-007, ADR-017.

## Open points

- **Maximum upload size** for images in calendar threads (see `docs/FITSPHERE_COACH_WEBAPP.md` → Open questions).
- Malware scanning, and whether to strip image metadata (EXIF location data) from uploads.
- Retention and deletion rules for client photos; re-verify current Cloudflare pricing and free-tier limits before go-live.
