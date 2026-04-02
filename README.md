# Music Recommender Simulation

## Project Summary

This project simulates how a content-based music recommender works — the same
underlying idea used by Spotify's "Radio" and YouTube's "Up Next." A user
provides a taste profile (preferred genre, mood, and energy level) and the
system scores every song in the catalog against those preferences, then returns
the top-K matches with plain-English explanations.

---

## How The System Works

### Real-world inspiration

Platforms like Spotify combine two main techniques. **Collaborative filtering**
says "people who liked what you liked also loved this" — it uses crowd behavior
rather than song attributes. **Content-based filtering** says "this song sounds
like what you enjoy" — it compares measurable song features (tempo, energy,
mood) directly to your stated preferences. This simulation implements the
content-based approach, which is simpler to reason about and easier to explain.

### Features used

Each `Song` carries: `genre`, `mood`, `energy` (0–1), `tempo_bpm`, `valence`,
`danceability`, and `acousticness` (0–1).

A `UserProfile` stores: `favorite_genre`, `favorite_mood`, `target_energy`,
and `likes_acoustic` (bool).

### Algorithm Recipe (scoring a single song)

| Rule | Points |
|------|--------|
| Genre matches user's favorite | +2.0 |
| Mood matches user's favorite | +1.0 |
| Energy proximity `1.0 – |song_energy – target_energy|` | 0.0 – 1.0 |
| Acousticness bonus (if `likes_acoustic`) `acousticness × 0.5` | 0.0 – 0.5 |

Genre is weighted highest because it is the broadest filter — a pop fan and a
metal fan rarely share the same "vibe" regardless of energy.

### Ranking rule

After every song is scored, the catalog is sorted from highest to lowest score
and the top-K entries are returned. `sorted()` is used (over `.sort()`) so the
original song list is never mutated.

### Data flow

```
Input (UserProfile)
    │
    ▼
For every Song in songs.csv
    │  score_song(user_prefs, song) → (numeric score, reasons string)
    │
    ▼
Sort all (song, score, reasons) by score descending
    │
    ▼
Output: Top-K recommendations with explanations
```

---

## Getting Started

### Setup

```bash
python3 -m venv .venv
source .venv/bin/activate   # Mac / Linux
pip install -r requirements.txt
```

### Run

```bash
python3 -m src.main
```

### Tests

```bash
pytest
```

---

## Experiments You Tried

### Experiment 1 – Energy-gap dominance

Profile: High-Energy Pop (energy=0.85). "Gym Hero" (pop/intense, energy=0.93)
scored 2.92 even without a mood match, while "Block Party Anthem" (hip-hop/happy,
energy=0.80) scored 1.95 without a genre match. This shows that when genre
matches but mood doesn't, energy proximity alone keeps the score competitive.

### Experiment 2 – Weight shift (doubled energy, halved genre)

Temporarily set genre weight to 1.0 and energy proximity weight to ×2. The
Chill Lofi profile then surfaced "Acoustic Confession" (folk/sad) near the top
because its energy (0.25) is very close to the target (0.38), even though it is
neither lofi nor chill. This confirmed that genre weight of 2.0 is important for
meaningful genre-level filtering.

### Experiment 3 – Adversarial profile (conflicting energy + mood)

Profile: `energy=0.9, mood=chill`. The system returned high-energy songs at the
top (because energy proximity dominated) but none of them were truly "chill"
— exposing the system's inability to understand that calm and high-energy are
semantically contradictory.

---

## Limitations and Risks

- Catalog has only 24 songs, so "diverse" recommendations are quickly exhausted.
- Genre weight can create a filter bubble: a pop user will rarely see jazz, even
  if the mood and energy are identical.
- The system treats every user as if their preferences are stable over time (no
  listening history, no feedback loop).
- Valence, danceability, and tempo_bpm are loaded but not used in scoring,
  leaving information on the table.

---

## Reflection

See [model_card.md](model_card.md) for the full model card and personal
reflection.

---

## Terminal Output Screenshots

### High-Energy Pop

```
Profile: High-Energy Pop  |  genre=pop, mood=happy, energy=0.85
#1  Sunrise City by Neon Echo           Score: 3.97  — genre match; mood match; energy similarity +0.97
#2  Gym Hero by Max Pulse               Score: 2.92  — genre match; energy similarity +0.92
#3  Block Party Anthem by Kilo Verse    Score: 1.95  — mood match; energy similarity +0.95
#4  Rooftop Lights by Indigo Parade     Score: 1.91  — mood match; energy similarity +0.91
#5  Golden Hour Soul by Velvet James    Score: 1.83  — mood match; energy similarity +0.83
```

### Chill Lofi

```
Profile: Chill Lofi  |  genre=lofi, mood=chill, energy=0.38
#1  Library Rain by Paper Lanterns      Score: 4.40  — genre; mood; energy +0.97; acoustic +0.43
#2  Midnight Coding by LoRoom           Score: 4.31  — genre; mood; energy +0.96; acoustic +0.35
#3  Focus Flow by LoRoom                Score: 3.37  — genre; energy +0.98; acoustic +0.39
#4  Mountain Trail Song by Cedar Folk   Score: 2.39  — mood; energy +0.95; acoustic +0.44
#5  Spacewalk Thoughts by Orbit Bloom   Score: 2.36  — mood; energy +0.90; acoustic +0.46
```

### Deep Intense Rock

```
Profile: Deep Intense Rock  |  genre=rock, mood=intense, energy=0.92
#1  Storm Runner by Voltline            Score: 3.99  — genre; mood; energy +0.99
#2  Solar Flare by Voltline             Score: 3.96  — genre; mood; energy +0.96
#3  Gym Hero by Max Pulse               Score: 1.99  — mood; energy +0.99
#4  Drop It Low by Circuit Breaker      Score: 1.97  — mood; energy +0.97
#5  Iron Sky by The Forge               Score: 1.96  — mood; energy +0.96
```

### Rainy Jazz Mood

```
Profile: Rainy Jazz Mood  |  genre=jazz, mood=moody, energy=0.32
#1  Jazz in the Rain by Slow Stereo     Score: 4.40  — genre; mood; energy +0.99; acoustic +0.41
#2  Coffee Shop Stories by Slow Stereo  Score: 3.40  — genre; energy +0.95; acoustic +0.45
#3  Neon Carousel by Dreamwave          Score: 1.72  — mood; energy +0.62; acoustic +0.10
#4  Night Drive Loop by Neon Echo       Score: 1.68  — mood; energy +0.57; acoustic +0.11
#5  Mountain Trail Song by Cedar Folk   Score: 1.43  — energy +0.99; acoustic +0.44
```

### Hip-Hop Focus

```
Profile: Hip-Hop Focus  |  genre=hip-hop, mood=focused, energy=0.76
#1  Pulse Check by Metro Grid           Score: 3.99  — genre; mood; energy +0.99
#2  Block Party Anthem by Kilo Verse    Score: 2.96  — genre; energy +0.96
#3  Focus Flow by LoRoom                Score: 1.64  — mood; energy +0.64
#4  Rooftop Lights by Indigo Parade     Score: 1.00  — energy +1.00
#5  Night Drive Loop by Neon Echo       Score: 0.99  — energy +0.99
```
