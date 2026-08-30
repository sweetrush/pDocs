## Automating Instagram Posts from the Command Line (Facebook Graph API)

### Introduction

Instagram has no official "post from your desktop app" API. What it has is the
**Instagram Graph API**, part of Meta's wider Graph API, which lets a server or
a script publish on behalf of an Instagram **Business** or **Creator** account.

This guide covers driving that API from the terminal with `curl`, wrapping it
in a script, scheduling it so posts go out unattended, and the neighbouring
Graph API capabilities worth knowing about (Facebook Page posting, native
scheduling, insights, batch requests).

### Read This First: Six Constraints That Shape Everything

Most failed Instagram automation projects die on one of these. Read them
before you write any code.

1. **Your media must already be at a public HTTPS URL.** You cannot upload a
   local file. `curl -F "file=@photo.jpg"` does not exist for Instagram feed
   posts - you pass `image_url=https://...` and Meta's servers fetch it. So you
   need somewhere to host media first: S3, Cloudflare R2, a web server, any
   public bucket. Plan for this; it is the single most common surprise.
2. **Personal accounts cannot use this at all.** The account must be converted
   to Business or Creator. There is no workaround.
3. **Instagram has no native scheduling in the API.** Unlike Facebook Pages,
   which accept a `scheduled_publish_time`, Instagram publishes *immediately*
   when you call `media_publish`. "Scheduled posting" means you run your own
   scheduler and call the API at the right moment - see section 11.
4. **Images must be JPEG.** PNG is not accepted for feed images. Convert first.
5. **Publishing to an account you do not own requires App Review.** Your own
   account works in development mode; publishing for clients means submitting
   your app for review, with a screencast per permission.
6. **Containers expire after 24 hours.** A container you create but never
   publish is dead after a day, and you start over.

> **On API versions.** Meta ships a new Graph API version roughly quarterly and
> deprecates old ones on a schedule. Every example here uses a
> `${GRAPH_VERSION}` variable rather than hardcoding - set it to the current
> version and check Meta's changelog before assuming an unversioned call works.

---

## 1. Which Instagram API You Actually Need

Meta ships **two different configurations** with different hosts, different
permission names, and different token mechanics. Picking the wrong one means
following setup steps that will never work for your case.

| | **Instagram API with Facebook Login** | **Instagram API with Instagram Login** |
|---|---|---|
| Host | `graph.facebook.com` | `graph.instagram.com` |
| Facebook Page required | **Yes** - linked to the IG account | No |
| Permissions | `instagram_basic`, `instagram_content_publish`, `pages_show_list`, `pages_read_engagement` | `instagram_business_basic`, `instagram_business_content_publish` |
| Business Manager integration | Yes | Limited |
| Best for | Agencies, multi-account, anything also touching Facebook Pages | A single self-owned account, simpler consumer-style login |

**This guide uses the Facebook Login path** (`graph.facebook.com`). It is the
long-established route, it integrates with Business Manager for managing
multiple client accounts, and it is what you need if you also want the Facebook
Page capabilities in section 13.

> Sources disagree on whether the Instagram Login path supports publishing -
> some claim it does not. That claim appears to be stale, conflating it with
> the retired *Instagram Basic Display API*, which genuinely had no publishing.
> If you choose that path, verify against current documentation rather than
> trusting a blog post, this one included.

---

## 2. Prerequisites

You need all five of these before a single API call will succeed:

- [ ] An Instagram account converted to **Business** or **Creator**
- [ ] A **Facebook Page**, with the Instagram account linked to it
- [ ] A **Meta Business account** (Business Manager)
- [ ] A **Meta developer app** with the *Instagram Graph API* product added
- [ ] Somewhere to host media at a public HTTPS URL

### Linking the Accounts

In the Instagram mobile app: **Settings → Account type and tools → Switch to
professional account**, then link it to your Facebook Page.

Verify the link from the Facebook side: **Page → Settings → Linked accounts →
Instagram**. If the Page does not show the Instagram account here, no amount of
API work will help - fix this first.

---

## 3. Getting a Long-Lived Access Token

Tokens are where most people get stuck, because there are several kinds and the
short-lived one you get first expires in about an hour.

### 3.1 Short-Lived Token

Open the [Graph API Explorer](https://developers.facebook.com/tools/explorer/),
select your app, and request these permissions:

```
instagram_basic
instagram_content_publish
pages_show_list
pages_read_engagement
business_management
```

Generate the token and copy it. **This expires in roughly one hour** - it is
only a stepping stone.

### 3.2 Exchange for a Long-Lived Token (~60 days)

```bash
GRAPH_VERSION="v23.0"
APP_ID="your-app-id"
APP_SECRET="your-app-secret"
SHORT_TOKEN="the-token-from-the-explorer"

curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/oauth/access_token" \
  -d "grant_type=fb_exchange_token" \
  -d "client_id=${APP_ID}" \
  -d "client_secret=${APP_SECRET}" \
  -d "fb_exchange_token=${SHORT_TOKEN}"
```

Returns an `access_token` valid for about 60 days.

### 3.3 Get a Page Access Token

```bash
LONG_TOKEN="the-long-lived-user-token"

curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/me/accounts" \
  -d "access_token=${LONG_TOKEN}"
```

Each Page in the response carries its own `access_token`. A Page token derived
from a **long-lived user token** does not expire on a fixed schedule, which is
what you want for automation.

### 3.4 Better for Production: a System User Token

For anything unattended, create a **System User** in Business Manager, assign
it to the app and the Page, and generate a token there. System User tokens can
be issued without expiry and are not tied to an employee's personal account -
so the automation does not break when someone leaves or changes their password.

> A token that can publish to your brand's Instagram is a credential of real
> consequence. Treat it like a production database password - see section 14.

---

## 4. Find Your Instagram User ID

Every publishing call needs the IG user ID, which is **not** your @handle.

```bash
PAGE_ID="your-page-id"
PAGE_TOKEN="your-page-access-token"

curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${PAGE_ID}" \
  -d "fields=instagram_business_account" \
  -d "access_token=${PAGE_TOKEN}"
```

Response:

```json
{
  "instagram_business_account": { "id": "17841400000000000" },
  "id": "1234567890"
}
```

That `id` is your `IG_USER_ID`. If the field comes back empty, the Page and
Instagram account are not properly linked - back to section 2.

---

## 5. Publishing a Single Image

The core flow is two calls: **create a container**, then **publish it**.

### Step 1 - Create the Container

```bash
IG_USER_ID="17841400000000000"
PAGE_TOKEN="your-page-access-token"

curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
  -d "image_url=https://example.com/media/launch.jpg" \
  -d "caption=New write-up is live. #docs #devops" \
  -d "access_token=${PAGE_TOKEN}"
```

Returns a container ID:

```json
{ "id": "17999999999999999" }
```

### Step 2 - Publish It

```bash
CREATION_ID="17999999999999999"

curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media_publish" \
  -d "creation_id=${CREATION_ID}" \
  -d "access_token=${PAGE_TOKEN}"
```

Returns the published media ID. The post is live immediately.

### Image Requirements

| Property | Requirement |
|---|---|
| Format | **JPEG only** - PNG is rejected |
| Hosting | Public HTTPS URL that Meta's servers can fetch |
| Aspect ratio | Between 4:5 and 1.91:1 |
| Caption | Up to 2,200 characters |
| Hashtags | Up to 30 |

Convert PNG before uploading:

```bash
# ImageMagick - flatten transparency onto white, since JPEG has no alpha
magick input.png -background white -alpha remove -alpha off -quality 88 output.jpg
```

---

## 6. Video and Reels

Video needs a **third step**: the container is processed asynchronously, and
publishing before processing finishes will fail.

### Step 1 - Create a REELS Container

```bash
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
  -d "media_type=REELS" \
  -d "video_url=https://example.com/media/demo.mp4" \
  -d "caption=Walkthrough of the new setup" \
  -d "access_token=${PAGE_TOKEN}"
```

### Step 2 - Poll Until Processing Finishes

```bash
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${CREATION_ID}" \
  -d "fields=status_code,status" \
  -d "access_token=${PAGE_TOKEN}"
```

`status_code` values:

| Value | Meaning |
|---|---|
| `IN_PROGRESS` | Still processing - keep waiting |
| `FINISHED` | Ready to publish |
| `ERROR` | Processing failed; read `status` for the reason |
| `EXPIRED` | Container was not published within 24 hours |
| `PUBLISHED` | Already published |

A polling loop with backoff:

```bash
poll_container() {
    local container_id="$1"
    local wait=10
    local elapsed=0
    local max=600   # give up after 10 minutes

    while [ "$elapsed" -lt "$max" ]; do
        local status
        status=$(curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${container_id}" \
                    -d "fields=status_code" \
                    -d "access_token=${PAGE_TOKEN}" | jq -r '.status_code')

        case "$status" in
            FINISHED) echo "ready"; return 0 ;;
            ERROR|EXPIRED) echo "failed: $status" >&2; return 1 ;;
        esac

        sleep "$wait"
        elapsed=$((elapsed + wait))
        # Back off gradually, capped - polling hard burns your hourly quota
        wait=$(( wait < 30 ? wait + 5 : 30 ))
    done

    echo "timed out waiting for container" >&2
    return 1
}
```

> Do not poll in a tight loop. Reel processing typically takes anywhere from a
> few seconds to a minute, and aggressive polling eats the hourly call quota
> you need for actual publishing. Ten seconds is a sane floor.

### Step 3 - Publish

Same `media_publish` call as for images.

---

## 7. Carousels

A carousel is *n+1* containers: one per image, plus a parent.

```bash
# 1. A child container per asset - note is_carousel_item=true
child1=$(curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
    -d "image_url=https://example.com/media/1.jpg" \
    -d "is_carousel_item=true" \
    -d "access_token=${PAGE_TOKEN}" | jq -r '.id')

child2=$(curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
    -d "image_url=https://example.com/media/2.jpg" \
    -d "is_carousel_item=true" \
    -d "access_token=${PAGE_TOKEN}" | jq -r '.id')

# 2. The parent container - caption goes here, not on the children
parent=$(curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
    -d "media_type=CAROUSEL" \
    -d "children=${child1},${child2}" \
    -d "caption=Swipe for the full walkthrough" \
    -d "access_token=${PAGE_TOKEN}" | jq -r '.id')

# 3. Publish the parent
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media_publish" \
    -d "creation_id=${parent}" \
    -d "access_token=${PAGE_TOKEN}"
```

Carousels take **2 to 10 items**. The caption belongs on the parent; captions
on children are ignored.

---

## 8. Stories

```bash
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/media" \
  -d "media_type=STORIES" \
  -d "image_url=https://example.com/media/story.jpg" \
  -d "access_token=${PAGE_TOKEN}"
```

Then publish as usual. Stories take no caption and expire after 24 hours.

---

## 9. Rate Limits

### The Publishing Limit

Instagram caps API-published posts per account over a rolling 24-hour window.
**Do not hardcode a number for this.** Published figures vary - 25, 50 and 100
all circulate, Meta's own documentation has been inconsistent, and the value
has changed more than once.

Query it instead:

```bash
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/content_publishing_limit" \
  -d "fields=config,quota_usage" \
  -d "access_token=${PAGE_TOKEN}"
```

```json
{
  "data": [{
    "config": { "quota_total": 100, "quota_duration": 86400 },
    "quota_usage": 12
  }]
}
```

`config.quota_total` is authoritative for *your* account right now. Build your
scheduler against this endpoint, checking before each publish, and you are
immune to the number changing under you.

### Application Call Limits

Separately, your app has an hourly call budget across all endpoints. Container
polling is the usual culprit for burning it - which is why section 6 backs off
rather than hammering.

---

## 10. A Complete Auto-Post Script

Putting it together, with the quota check and error handling that separate a
demo from something you can leave running.

```bash
#!/usr/bin/env bash
#
# ig-post.sh - publish an image to Instagram via the Graph API.
# Usage: ./ig-post.sh <public-image-url> "<caption>"

set -euo pipefail

: "${GRAPH_VERSION:=v23.0}"
: "${IG_USER_ID:?IG_USER_ID must be set}"
: "${PAGE_TOKEN:?PAGE_TOKEN must be set}"

API="https://graph.facebook.com/${GRAPH_VERSION}"
IMAGE_URL="${1:?usage: ig-post.sh <image-url> <caption>}"
CAPTION="${2:-}"

# Fail fast on a Graph API error object rather than continuing with junk
check_error() {
    local body="$1" step="$2"
    if echo "$body" | jq -e '.error' >/dev/null 2>&1; then
        echo "[$step] Graph API error:" >&2
        echo "$body" | jq '.error | {message, type, code, error_subcode}' >&2
        exit 1
    fi
}

# 1. Refuse to start if we are at the publishing quota
limit=$(curl -sfG "${API}/${IG_USER_ID}/content_publishing_limit" \
            -d "fields=config,quota_usage" \
            -d "access_token=${PAGE_TOKEN}")
check_error "$limit" "quota"

used=$(echo "$limit"  | jq -r '.data[0].quota_usage // 0')
total=$(echo "$limit" | jq -r '.data[0].config.quota_total // 0')

if [ "$total" -gt 0 ] && [ "$used" -ge "$total" ]; then
    echo "Publishing quota exhausted (${used}/${total}); try again later." >&2
    exit 75   # EX_TEMPFAIL - a retrying scheduler can treat this as soft
fi
echo "Quota: ${used}/${total}"

# 2. Create the container
container=$(curl -sf -X POST "${API}/${IG_USER_ID}/media" \
    --data-urlencode "image_url=${IMAGE_URL}" \
    --data-urlencode "caption=${CAPTION}" \
    -d "access_token=${PAGE_TOKEN}")
check_error "$container" "container"

creation_id=$(echo "$container" | jq -r '.id')
echo "Container: ${creation_id}"

# 3. Publish
published=$(curl -sf -X POST "${API}/${IG_USER_ID}/media_publish" \
    -d "creation_id=${creation_id}" \
    -d "access_token=${PAGE_TOKEN}")
check_error "$published" "publish"

media_id=$(echo "$published" | jq -r '.id')
echo "Published: ${media_id}"
```

Note `--data-urlencode` for the caption and URL. A caption containing `&`, `#`
or a newline will silently corrupt the request with plain `-d`, and hashtags
make that likely.

Run it:

```bash
chmod +x ig-post.sh
export IG_USER_ID="17841400000000000"
export PAGE_TOKEN="$(pass show meta/ig-page-token)"   # see section 14
./ig-post.sh "https://example.com/media/launch.jpg" "New guide is live #devops"
```

---

## 11. Scheduling: Making It an "Auto" Post

Instagram publishes immediately, so scheduling is entirely your side.

### cron

```bash
crontab -e
```

```
# Post daily at 09:00, sourcing env from a private file
0 9 * * * . $HOME/.config/ig/env && $HOME/bin/ig-post.sh "$(cat $HOME/queue/next-url)" "$(cat $HOME/queue/next-caption)" >> $HOME/log/ig.log 2>&1
```

Note cron does **not** load your shell profile, so the environment file must be
sourced explicitly - a very common reason a script that works interactively
does nothing from cron.

### systemd Timer

More robust, with real logging and retry:

```ini
# ~/.config/systemd/user/ig-post.service
[Unit]
Description=Publish queued Instagram post

[Service]
Type=oneshot
EnvironmentFile=%h/.config/ig/env
ExecStart=%h/bin/ig-post.sh
```

```ini
# ~/.config/systemd/user/ig-post.timer
[Unit]
Description=Daily Instagram post

[Timer]
OnCalendar=*-*-* 09:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl --user enable --now ig-post.timer
systemctl --user list-timers ig-post
journalctl --user -u ig-post -f
```

`Persistent=true` catches up a missed run if the machine was off - usually what
you want for a daily post.

---

## 12. Parallel and Batch Posting

If you are posting for several accounts, or clearing a queue, the temptation is
to fan out with `xargs -P` or `parallel`. Some care is needed.

**What parallelises safely:** operations across **different** IG accounts, and
creating carousel *child* containers for one post.

**What does not:** multiple `media_publish` calls to the *same* account. Publish
order is not guaranteed, you can trip the publishing quota mid-flight, and a
partially-published carousel is not recoverable. Publish serially per account.

Creating carousel children concurrently, then publishing once:

```bash
# Children are independent - safe to create in parallel
export API GRAPH_VERSION IG_USER_ID PAGE_TOKEN
children=$(printf '%s\n' \
        "https://example.com/1.jpg" \
        "https://example.com/2.jpg" \
        "https://example.com/3.jpg" |
    xargs -P 3 -I{} sh -c '
        curl -sf -X POST "${API}/${IG_USER_ID}/media" \
            --data-urlencode "image_url={}" \
            -d "is_carousel_item=true" \
            -d "access_token=${PAGE_TOKEN}" | jq -r ".id"
    ' | paste -sd, -)
```

> Order matters for carousels and `xargs -P` does not preserve it. If the
> sequence of slides is meaningful, either create children serially or tag and
> re-sort the results before building the `children` list.

For multiple accounts, cap concurrency and keep each account serial:

```bash
# One worker per account; each account's posts stay in order
cat accounts.txt | xargs -P 4 -I{} ./post-for-account.sh {}
```

### Graph API Batch Requests

Meta also supports batching up to 50 calls in one HTTP request:

```bash
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/" \
  -d "access_token=${PAGE_TOKEN}" \
  --data-urlencode 'batch=[
    {"method":"GET","relative_url":"me?fields=id,name"},
    {"method":"GET","relative_url":"'"${IG_USER_ID}"'?fields=username,followers_count"}
  ]'
```

Useful for reads. It does **not** exempt you from rate limits - each sub-request
still counts - and it is a poor fit for publishing, where you need each step's
result before the next.

---

## 13. Other Things the Graph API Can Do

Publishing to Instagram is one corner of the Graph API. The same app, token and
Page unlock a good deal more.

### Facebook Page Posting - Including Native Scheduling

Unlike Instagram, Facebook Pages **do** support server-side scheduling:

```bash
# Post to a Page immediately
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${PAGE_ID}/feed" \
  --data-urlencode "message=New write-up published" \
  --data-urlencode "link=https://example.com/post" \
  -d "access_token=${PAGE_TOKEN}"

# Or schedule it - Unix timestamp, 10 minutes to 6 months out
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${PAGE_ID}/feed" \
  --data-urlencode "message=Goes out tomorrow morning" \
  -d "published=false" \
  -d "scheduled_publish_time=$(date -d 'tomorrow 09:00' +%s)" \
  -d "access_token=${PAGE_TOKEN}"
```

Requires `pages_manage_posts`. If cross-posting the same content, note the
asymmetry: Facebook can hold it for you, Instagram cannot.

### Photos to a Page

Facebook Pages *do* accept a direct file upload, unlike Instagram:

```bash
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${PAGE_ID}/photos" \
  -F "source=@./local-photo.jpg" \
  -F "caption=Uploaded straight from disk" \
  -F "access_token=${PAGE_TOKEN}"
```

### Insights and Analytics

```bash
# Account-level Instagram metrics
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}/insights" \
  -d "metric=impressions,reach,profile_views" \
  -d "period=day" \
  -d "access_token=${PAGE_TOKEN}"

# Per-post metrics
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${MEDIA_ID}/insights" \
  -d "metric=engagement,impressions,reach,saved" \
  -d "access_token=${PAGE_TOKEN}"
```

Requires `instagram_manage_insights`. Metric names change between API versions
more often than most fields - check the changelog when a metric stops returning.

### Comment Management

```bash
# Read comments on a post
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${MEDIA_ID}/comments" \
  -d "fields=id,text,username,timestamp" \
  -d "access_token=${PAGE_TOKEN}"

# Reply
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${COMMENT_ID}/replies" \
  --data-urlencode "message=Thanks for reading!" \
  -d "access_token=${PAGE_TOKEN}"

# Hide rather than delete
curl -sX POST "https://graph.facebook.com/${GRAPH_VERSION}/${COMMENT_ID}" \
  -d "hide=true" \
  -d "access_token=${PAGE_TOKEN}"
```

Requires `instagram_manage_comments`.

### Webhooks

Rather than polling for new comments or mentions, subscribe a Page to webhooks
and have Meta POST events to your endpoint. Far cheaper against your call quota
than a polling loop, and the right design once you are past a handful of posts.

### Other Surfaces on the Same Graph

- **Media discovery** - `/{ig-user-id}?fields=media{caption,like_count}` for
  your own back catalogue
- **Hashtag search** - find recent public media for a hashtag
  (`instagram_manage_insights`, with tight limits)
- **Mentions** - respond where the account is tagged
- **Messenger / Instagram Direct** - separate permissions and review

---

## 14. Handling the Token Safely

A Page token that can publish is a credential. Some rules:

**Never put it in the script, or in the repository.** Use an environment file
with tight permissions, or a secret manager:

```bash
mkdir -p ~/.config/ig
cat > ~/.config/ig/env <<'EOF'
IG_USER_ID=17841400000000000
PAGE_TOKEN=your-page-access-token
GRAPH_VERSION=v23.0
EOF
chmod 600 ~/.config/ig/env
```

Add it to `.gitignore` and confirm:

```bash
echo '.config/ig/env' >> .gitignore
git check-ignore -v ~/.config/ig/env   # should print a match
```

**Never pass the token as a command-line argument on a shared host.** Arguments
are visible in `ps` to other users; environment variables are not, to the same
degree. This is why the script above reads `PAGE_TOKEN` from the environment.

**Beware it landing in logs.** `curl -v`, shell tracing (`set -x`), and CI logs
will all happily print your token. In CI, register it as a masked secret.

**Prefer a System User token** for automation (section 3.4), and rotate it if
it is ever exposed - regenerate in Business Manager, which invalidates the old
one.

**Use `appsecret_proof`** for server-side calls to harden against a stolen
token being replayed from elsewhere:

```bash
proof=$(printf '%s' "${PAGE_TOKEN}" | openssl dgst -sha256 -hmac "${APP_SECRET}" | awk '{print $2}')
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/${IG_USER_ID}" \
  -d "fields=username" \
  -d "access_token=${PAGE_TOKEN}" \
  -d "appsecret_proof=${proof}"
```

**Scope the permissions you request.** If the automation only publishes, do not
also request comment and insights permissions - each one you hold is additional
blast radius, and each one needs separate App Review justification anyway.

---

## 15. Troubleshooting

Inspect what a token actually is and what it can do:

```bash
curl -sG "https://graph.facebook.com/${GRAPH_VERSION}/debug_token" \
  -d "input_token=${PAGE_TOKEN}" \
  -d "access_token=${APP_ID}|${APP_SECRET}"
```

This returns the token type, the app it belongs to, its expiry, and its
granted scopes - which resolves most "why is this failing" questions in one
call.

| Symptom | Likely cause |
|---|---|
| `(#10) Application does not have permission` | Missing permission, or App Review not approved for this action |
| `(#100) Invalid parameter` on container creation | Media URL not publicly reachable, or PNG passed where JPEG required |
| `instagram_business_account` field empty | Instagram account not linked to the Page, or not a Business/Creator account |
| `(#4) Application request limit reached` | Hourly call quota exhausted - usually aggressive polling |
| `The user is not an Instagram Business` | Account still personal - convert it |
| Container stuck `IN_PROGRESS` forever | Video fails Meta's spec (codec, duration, size); re-encode |
| `status_code: EXPIRED` | Container older than 24h; create a new one |
| Works for your account, fails for a client's | Development mode - needs App Review to act on accounts you do not own |

**Verify a media URL is genuinely reachable** the way Meta will fetch it -
from the public internet, not your laptop:

```bash
curl -sI "https://example.com/media/launch.jpg" | head -n 1
# Expect: HTTP/2 200 - a 403, a redirect to a login page, or a private
# bucket URL will all fail silently on Meta's side
```

---

## 16. Resources

### Official Documentation

- **Instagram Platform**: https://developers.facebook.com/docs/instagram-platform
- **Content Publishing**: https://developers.facebook.com/docs/instagram-platform/content-publishing
- **Graph API Changelog**: https://developers.facebook.com/docs/graph-api/changelog
- **Graph API Explorer**: https://developers.facebook.com/tools/explorer/
- **Access Token Debugger**: https://developers.facebook.com/tools/debug/accesstoken/

### Related Guides in This Repository

- [Web Development Manual](Webdevmanual.md) - general web development reference
- [Advanced SSH Connectivity](../LinuxWriteUps/EnhancedSSH_Connectivity.md) -
  running scheduled jobs on a remote host
- [Tailscale Mesh VPN](../SecArea/Tailscale_Windows_Setup.md) - reaching the
  box that runs your scheduler without exposing it

---

## Summary

This guide covered:

- The two Instagram API configurations and how to pick one
- Account linking, App setup, and the token chain from short-lived to System
  User
- The container-then-publish flow for images, Reels, carousels and stories
- Polling container status with backoff
- Querying the publishing quota rather than hardcoding it
- A complete, quota-aware posting script and two ways to schedule it
- Where parallelism is safe and where it corrupts posts
- Facebook Page posting, native scheduling, insights, comments and webhooks
- Token handling, `appsecret_proof`, and troubleshooting

**Key Takeaways:**

- Media must be at a public HTTPS URL - there is no local upload for Instagram
  feed posts, and this drives your whole architecture
- Instagram has no native scheduling; Facebook Pages do. Plan around it.
- Never hardcode the publishing limit - query `content_publishing_limit`
- Video needs the poll step; publishing before `FINISHED` fails
- Publish serially per account, even when creating containers in parallel
- Images are JPEG only, and a PNG produces an unhelpful generic error
- App Review is required before you can publish for anyone but yourself

**Next Steps:**

1. Convert the account, link the Page, verify `instagram_business_account`
   returns an ID
2. Get a long-lived token and publish one image by hand with `curl`
3. Wrap it in the script and confirm the quota check works
4. Add a scheduler and let one real post go out unattended
5. Submit for App Review before promising a client anything

Remember: automated posting is still posting. Rate limits and App Review exist
because the platform treats aggressive automation as abuse - keep volumes
human, and do not automate engagement (bulk follows, generic comments) that
would put the account at risk of restriction.
