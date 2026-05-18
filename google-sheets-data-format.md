# Google Sheets Data Format

The Google Sheets spreadsheet is the raw data source for the Grafana dashboard. It uses a long/tidy format optimised for SQL querying — no totals, formatting, or computed columns. All aggregation happens in Grafana.

## Votes Schema

One row per points award. Only rows where points were actually awarded are needed — no zero or blank rows.

| Column | Type | Description |
|--------|------|-------------|
| `voting_country` | string | The country casting the vote, or `Rest of World`. Includes all 35 participating countries — both finalists and countries eliminated in the semi-finals — as all participate in the final vote. |
| `recipient_country` | string | The country whose song is receiving the points. Only the 26 finalists appear here — eliminated countries cannot receive votes. |
| `vote_type` | string | Either `jury` or `televote` |
| `points` | integer | Points awarded: one of 1, 2, 3, 4, 5, 6, 7, 8, 10, 12 |

## Example Rows

| voting_country | recipient_country | vote_type | points |
|----------------|-------------------|-----------|--------|
| France | Italy | jury | 12 |
| France | Sweden | jury | 10 |
| France | Italy | televote | 8 |
| Rest of World | Ukraine | televote | 12 |

## Volume

With 36 voting countries (35 participating countries + Rest of World), 2 vote types each, and up to 10 recipients per voting country per type, the sheet will have a maximum of **720 rows**. Note that Rest of World casts a televote only, so the true maximum is 710 rows.

## Example Grafana Queries

```sql
-- Overall ranking
SELECT recipient_country, SUM(points) AS total
FROM votes
GROUP BY recipient_country
ORDER BY total DESC

-- Jury vs televote split for a given song
SELECT vote_type, SUM(points) AS total
FROM votes
WHERE recipient_country = 'Italy'
GROUP BY vote_type

-- Which countries awarded 12 points to a given song
SELECT voting_country, vote_type
FROM votes
WHERE recipient_country = 'Italy' AND points = 12
```

## Data Entry

Rows are appended as results are announced during the show. Each voting country announces jury points live, with televote points aggregated and revealed separately — so jury rows and televote rows will be entered in two distinct phases during the evening.

## Running Order Schema

A separate sheet (`running-order.csv`) lists the finalists in performance order along with a YouTube video ID for each song. One row per finalist.

| Column | Type | Description |
|--------|------|-------------|
| `running_order` | integer | Position in the running order of the final (1 = performs first). Numbers may be non-contiguous if countries are eliminated after the running order is drawn — gaps preserve the originally drawn positions. |
| `country` | string | The participating country. Matches values used in `recipient_country` in the votes sheet. |
| `artist_name` | string | The performing artist or group. |
| `song_title` | string | The title of the song being performed. |
| `youtube_id` | string | YouTube video ID for the country's official music video (the `v=` parameter from a YouTube URL). Prefix with `https://www.youtube.com/watch?v=` to construct a full URL, or `https://www.youtube.com/embed/` to embed. |

### Example Rows

| running_order | country | artist_name | song_title | youtube_id |
|---------------|---------|-------------|------------|------------|
| 1 | Luxembourg | Eva Marija | Mother Nature | DmVfJSRqgnI |
| 2 | Israel | Noam Bettan | Michelle | xWCnWSoG8nI |
| 4 | United Kingdom | Look Mum No Computer | Eins, Zwei, Drei | niMKvJ-Itq8 |
