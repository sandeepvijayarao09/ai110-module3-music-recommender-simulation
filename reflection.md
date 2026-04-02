# Reflection: Profile Comparison Notes

## High-Energy Pop vs. Chill Lofi

The High-Energy Pop profile (energy=0.85, genre=pop, mood=happy) pulled
"Sunrise City" to #1 with a score of 3.97 — a triple hit on genre, mood, and
energy. The Chill Lofi profile (energy=0.38, genre=lofi, mood=chill) pulled
"Library Rain" to #1 with 4.40 — slightly higher because `likes_acoustic=True`
added an acousticness bonus.

The key difference: the energy dimension flipped the entire ranking. Songs that
scored near-zero for Chill Lofi (like Gym Hero at energy=0.93) were competitive
in the Pop profile. This makes sense — energy is the most continuous feature in
the scoring, meaning a big energy mismatch penalizes a song almost as heavily as
a genre miss.

## Deep Intense Rock vs. Rainy Jazz Mood

Rock/intense (energy=0.92) and Jazz/moody (energy=0.32) sit at opposite ends of
the energy axis, so their top results share almost no overlap. The Rock profile
surfaced two Voltline tracks back-to-back (Storm Runner, Solar Flare) because
they are the only `rock` songs in the catalog and both match `intense` — showing
how a small catalog can exhaust genre diversity quickly.

The Jazz profile surfaced "Mountain Trail Song" (folk/chill) at #5 because its
energy (0.33) nearly perfectly matches the target (0.32). No genre or mood
match, but energy proximity alone put it in the top 5. This highlights that low
energy is acoustically correlated with certain non-jazz genres (folk, ambient,
classical) — the scoring logic cannot distinguish between them at the feature
level.

## Hip-Hop Focus vs. Deep Intense Rock

Both profiles want high energy (0.76 vs. 0.92) but in very different genres.
The Hip-Hop Focus profile ran out of genre matches after two songs (Pulse Check,
Block Party Anthem) and then filled #3–#5 with lofi and synthwave tracks purely
on energy. The Rock profile never left its genre+mood bubble because two rock
songs exist with near-perfect scores.

This comparison shows that genre coverage in the dataset matters as much as the
scoring weights: a genre with only 2 songs in the catalog will exhaust its
top-match pool faster than a genre with 5–6 songs.

## Adversarial Profile: High-Energy Chill

Profile tested: `genre="ambient", mood="chill", energy=0.9, likes_acoustic=True`

Expected behavior: calm, atmospheric songs. Actual result: "Iron Sky" (metal,
intense, energy=0.96) appeared at #3 purely due to energy proximity, despite
being the furthest possible match for a chill mood. The energy dimension
overwhelmed the mood signal. This is the clearest example of how a simple
weighted score cannot encode semantic contradictions.
