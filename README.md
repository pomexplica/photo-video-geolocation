# Photo & Video Geolocation - Where Was It Taken? Privacy Check

Find where a photo or video was taken, from a file or an Instagram, TikTok, YouTube, X or Facebook link: best-guess place, alternatives with radius, confidence, clues, GPS in the file, spoken language and electric hum. Plus a privacy report and a check of a claimed place.

**[Run it on Apify](https://apify.com/pomlabs/photo-video-geolocation?utm_source=github&utm_medium=readme&utm_campaign=photo-video-geolocation)**: no install, free credit on a new Apify account.

Find **where a photo or a video was taken**, from a file or straight from a **social media link** (Instagram, TikTok, YouTube, X, Facebook, Threads, Reddit, Kwai and 1,000+ sites). You get a best-guess place with coordinates and radius, the alternatives, a confidence level, the clues that point there, the **GPS hidden in the file**, the **spoken language** and the **electric hum** of the audio (50 or 60 Hz grid). Each result also comes with a **privacy report** (what in the image or file gives the place away, and how to fix it) and, if you want, a **check of a claimed place** ("was this really filmed in Lisbon?").

## Why this one

- **Links, not only files:** paste a Reel, a TikTok, a Short or a tweet. Videos are read on several frames, and their sound is checked too.
- **Two AI engines, and you see when they agree:** both look at the same frames. When they land within 25 km of each other and both are sure (radius up to 100 km), the result is marked `high`; otherwise both places come as candidates and the result is `medium` or `low`. No single overconfident pin.
- **What the file says, before any guess:** GPS coordinates, capture time and phone model from the photo's EXIF or the video's metadata. A GPS in the file wins over any guess.
- **Sound clues for videos:** the language spoken in the first minute, and the mains hum (50 Hz in Europe, Africa, most of Asia; 60 Hz in the Americas, Brazil included), which splits the world in two.
- **Privacy report:** every leak found (GPS, readable signs, house numbers, plates, landmarks, capture time, device), rated and with the fix. Made for creators and anyone who wants to check a photo before posting it.
- **Claim check:** give the place someone says it was taken; each result says `consistent`, `inconsistent` or `uncertain`, with the distance.
- **Nothing kept:** the photo or video is processed in memory and a temporary folder, and deleted at the end of each item.

## Accuracy (measured, so you know what to expect)

Tested on 25 street-level photos that are **not on any web page** (fresh Street View frames from 25 ordinary cities, 10 in Brazil and 15 elsewhere), so the engines could not have memorized them:

| | Within 1 km | Within 25 km | Within 200 km |
|---|---|---|---|
| Best of the two engines, all 25 | 6 | 15 | 20 |
| Cities outside Brazil (15) | 6 | 13 | 14 |
| Inland Brazil (10) | 0 | 2 | 6 |
| Results marked `high` (9) | 3 | 8 | 9 |

In plain words: on a street photo of a city, it usually gets the **city**; in small inland towns, it usually gets the **region**. Famous places and photos with readable signs do much better. Indoor shots, close-ups and selfies carry few clues: the result then says so (`status: "no_location"`) instead of inventing a place.

## Input

| Field | What it does |
|---|---|
| `urls` | Links to photos, videos or public posts. Up to 100 per run |
| `upload` | Or upload one photo or video |
| `claimedPlace` | Optional place to verify, like `Lisbon, Portugal` |
| `transcribeSpeech` | Detect the spoken language of videos (default on, included) |
| `videoFrames` | Frames per video, 1 to 3 (default 3) |

## Output

```json
{
  "type": "analysis",
  "input": "https://www.tiktok.com/@scout2015/video/6718335390845095173",
  "mediaType": "video",
  "platform": "tiktok",
  "author": "scout2015",
  "status": "ok",
  "confidence": "medium",
  "bestGuess": {"lat": 24.7136, "lon": -81.0876, "city": "Marathon", "country": "United States", "radiusKm": 350.0, "engine": "A"},
  "candidates": [
    {"lat": 24.7136, "lon": -81.0876, "city": "Marathon", "country": "United States", "radiusKm": 350.0, "engine": "A"},
    {"lat": 24.7, "lon": -81.1, "city": "Florida Keys (approx., e.g. Marathon/Key Colony Beach area)", "country": "United States", "radiusKm": 150.0, "engine": "B"}
  ],
  "enginesApartKm": 2.0,
  "clues": ["Thatched tiki-hut (chickee-style) roof beside a pool, typical of Florida and Caribbean resorts", "Pastel stucco, elevated beach-style house in the background, typical of the Florida Keys", "Low confidence: the scene could also fit the Bahamas or the Gulf Coast of Florida"],
  "visibleLeaks": ["Pastel buildings in the background of frame 1 could help identify the neighborhood if matched in street imagery"],
  "fileGps": null,
  "durationSeconds": 10.5,
  "privacy": {"risk": "medium", "items": [{"leak": "The scene suggests a region", "detail": "United States", "risk": "medium", "fix": "Avoid distinctive streets, signs and landmarks in the frame"}]},
  "seconds": 11.5
}
```

With `claimedPlace`, each result also has `claim`: `{"claimedPlace": "Lisbon, Portugal", "distanceKm": 0.4, "verdict": "consistent", "enginesSay": [...]}`.

Links that cannot be opened (private, deleted, not a photo or a video) come as a row with `type: "error"` and the reason, and are **not charged**.

## Use cases

- **Creators and influencers:** check a photo or a Reel before posting: does it reveal your home, your street or your kid's school?
- **OSINT, journalism and fact-checking:** where was this viral video filmed? Is the place in the caption true?
- **Trust & safety and marketplaces:** do the listing photos match the city the seller claims?
- **Travel and real estate content:** tag old photos and videos with a place.

## Pricing

- **$0.0225 per photo or video analyzed** (both engines, sound clues and the privacy report included). No charge per run, and no charge for links that fail.

## Notes

- The engines are AI models: treat the result as a strong lead with a radius, not as proof. A GPS in the file is the exact place where the camera was.
- Most social networks strip GPS from uploaded photos and videos; the original file from the phone usually still has it.
- One video is read in about 10 to 20 seconds; runs process 4 items at a time.

## Call it by API

```bash
curl -X POST "https://api.apify.com/v2/acts/pomlabs~photo-video-geolocation/run-sync-get-dataset-items?token=YOUR_APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://upload.wikimedia.org/wikipedia/commons/0/0f/0123_RUA_AUGUSTA_Lisbon.jpg"]}'
```

Python:

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("pomlabs/photo-video-geolocation").call(run_input={"urls": ["https://upload.wikimedia.org/wikipedia/commons/0/0f/0123_RUA_AUGUSTA_Lisbon.jpg"]})
for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(item)
```

## Use it from Claude, Cursor or any MCP client

```bash
claude mcp add photo-video-geolocation "https://mcp.apify.com?tools=pomlabs/photo-video-geolocation" -t http
```

## More Actors by Pom Labs

All our Actors: [apify.com/pomlabs](https://apify.com/pomlabs?utm_source=github&utm_medium=readme&utm_campaign=photo-video-geolocation)
