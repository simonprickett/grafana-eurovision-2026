# Google Sheets Data Format

The Google Sheets spreadsheet is the raw data source for the Grafana dashboard. It uses a long/tidy format optimised for SQL querying — no totals, formatting, or computed columns. All aggregation happens in Grafana.

## Schema

One row per points award. Only rows where points were actually awarded are needed — no zero or blank rows.

| Column | Type | Description |
|--------|------|-------------|
| `voting_bloc` | string | The country casting the vote, or `Rest of World` |
| `recipient_country` | string | The country whose song is receiving the points |
| `vote_type` | string | Either `jury` or `televote` |
| `points` | integer | Points awarded: one of 1, 2, 3, 4, 5, 6, 7, 8, 10, 12 |

## Example Rows

| voting_bloc | recipient_country | vote_type | points |
|-------------|-------------------|-----------|--------|
| France | Italy | jury | 12 |
| France | Sweden | jury | 10 |
| France | Italy | televote | 8 |
| Rest of World | Ukraine | televote | 12 |

## Volume

With 36 voting blocs (35 countries + Rest of World), 2 vote types each, and up to 10 recipients per bloc per type, the sheet will have a maximum of **720 rows**.

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

-- Which blocs awarded 12 points to a given song
SELECT voting_bloc, vote_type
FROM votes
WHERE recipient_country = 'Italy' AND points = 12
```

## Data Entry

Rows are appended as results are announced during the show. Each voting bloc announces jury points live, with televote points aggregated and revealed separately — so jury rows and televote rows will be entered in two distinct phases during the evening.
