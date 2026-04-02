# Model Card: Music Recommender Simulation

## 1. Model Name

**VibeFinder 1.0**

---

## 2. Intended Use

This model suggests up to 5 songs from a 24-song catalog based on a user's
preferred genre, mood, and energy level. It is designed for classroom
exploration of how content-based recommendation works — not for use in a
production music platform or for making decisions about real users.

**Non-intended uses:** real-time streaming recommendations, replacing
collaborative filtering, serving users with hearing or accessibility needs
without additional consideration.

---

## 3. How the Model Works

Think of it like a talent scout with a checklist. The scout looks at every song
in the catalog and asks three questions:

1. **Is the genre right?** If yes, the song gets 2 points.
2. **Is the mood right?** If yes, it gets 1 more point.
3. **How close is the energy level?** The closer to the user's target, the more
   the song earns — up to 1 full point for a perfect match.

Optionally, if the user likes acoustic-sounding music, songs with high
acousticness get a small bonus (up to 0.5 points).

Once every song is scored, the catalog is sorted from highest to lowest score
and the top 5 are returned with plain-English explanations.

---

## 4. Data

- **Catalog size:** 24 songs (10 from the starter file + 14 added)
- **Genres represented:** pop, lofi, rock, ambient, jazz, synthwave, indie pop,
  metal, country, r&b, classical, electronic, hip-hop, folk, reggae
- **Moods represented:** happy, chill, intense, relaxed, focused, moody, sad
- **Limitations:** All artists are fictional. There are no lyrics, no release
  year, no play counts, and no real listener data. The catalog was designed by
  one person and likely reflects a Western-centric musical vocabulary.

---

## 5. Strengths

- Works well for users with a clear, consistent taste profile — "Chill Lofi"
  and "Deep Intense Rock" consistently surfaced highly relevant results.
- Explanations are transparent: users see exactly why a song ranked where it did.
- Simple enough to audit — a non-programmer can follow the scoring logic after
  one reading.

---

## 6. Limitations and Bias

- **Filter bubble:** Genre weight of 2.0 means a pop fan will almost never see
  jazz or folk near the top, even if those songs are a near-perfect energy and
  mood match. This mirrors real-world filter bubbles on streaming platforms.
- **Pop over-representation:** Even after expanding the dataset, pop-adjacent
  genres (indie pop, r&b, hip-hop) collectively dominate, so pop-profile users
  get more variety than, say, classical users.
- **No diversity control:** If two songs have identical scores, the system may
  recommend both songs from the same artist back-to-back without variation.
- **Conflicting preferences unsupported:** A user who wants energy=0.9 and
  mood=chill gets high-energy results because energy proximity mathematically
  wins — the system cannot detect that "high-energy chill" is a contradiction.
- **No temporal context:** The system treats all songs equally regardless of
  recency or trending signals.

---

## 7. Evaluation

Five distinct user profiles were tested:

| Profile | Top Result | Matched Intuition? |
|---------|-----------|-------------------|
| High-Energy Pop | Sunrise City (pop/happy) | Yes — exact match |
| Chill Lofi | Library Rain (lofi/chill) | Yes — top 2 were lofi |
| Deep Intense Rock | Storm Runner (rock/intense) | Yes |
| Rainy Jazz Mood | Jazz in the Rain (jazz/moody) | Yes |
| Hip-Hop Focus | Pulse Check (hip-hop/focused) | Yes |

**Surprise finding:** The "Rainy Jazz Mood" profile recommended "Mountain Trail
Song" (folk/chill) at #5 purely on energy proximity (+0.99) and acousticness
bonus — not genre or mood. This is a sensible edge case: folk and jazz share
acoustic texture even if genre labels differ.

**Weight-shift experiment:** Doubling energy weight caused "Acoustic Confession"
(folk/sad) to break into the Chill Lofi top results — confirming that genre
weight is load-bearing for genre coherence.

---

## 8. Future Work

1. **Add a diversity penalty** — cap each artist at one appearance in the top-5.
2. **Use valence and danceability** — these features are currently loaded but
   never scored; incorporating them would better distinguish "euphoric pop" from
   "melancholy pop."
3. **Tempo range matching** — instead of a binary genre match, allow the user to
   specify a BPM range (e.g., 80–110) and score proximity to that range.
4. **Hybrid collaborative + content** — store implicit feedback (skips, replays)
   and blend it with the content score to improve cold-start handling.

---

## 9. Personal Reflection

Building VibeFinder made something abstract click: a "recommendation" is just a
sorted list produced by a scoring function you designed. The moment I saw the
Chill Lofi profile pull Mountain Trail Song (folk) into its top 5 purely on
energy proximity, I realized how real-world filter bubbles form — not from
malice, but from one weight being slightly too high.

Using AI tools during this project was genuinely useful for brainstorming the
scoring weights and generating diverse song data, but I had to verify the CSV
format and numeric types by reading the code myself — the AI occasionally
suggested float comparisons that would break on string inputs. That taught me
the most important rule: understand the code before you ship it, regardless of
where it came from.

What surprised me most is that simple arithmetic — a few additions and an
absolute-value subtraction — can already "feel" like taste. That feeling is
partly a cognitive trick: we see our favorite genre at the top and assume the
system "understands" us. Real understanding would require context, history, and
listening to the actual audio. VibeFinder has none of that — and that gap is
exactly where human judgment still matters most.
