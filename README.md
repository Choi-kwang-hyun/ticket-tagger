# Ticket Tagger
Machine learning driven issue classification bot.
[Add to your repository now!](https://github.com/apps/ticket-tagger/installations/new)

![](https://thumbs.gfycat.com/GreedyBrownHochstettersfrog-size_restricted.gif)

### License

```
Ticket Tagger automatically predicts and labels issue types.
Copyright (C) 2018,2019,2020  Rafael Kallis

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>. 
```

### Development

#### notice:

- nodejs `^12.x` is required to compile/install dependencies
- `wget` is required for fetching datasets

#### get started:

```sh
git clone https://github.com/rafaelkallis/ticket-tagger ticket-tagger
cd ticket-tagger

# install appropriate nodejs version
# nvm install 12
# nvm use 12
npx nave use 12

# compile/install dependencies
npm install

# fetch dataset
npm run dataset

# run benchmark
npm run benchmark

# run linter
npm run lint

# run tests
npm test

# run server
npm start
```

#### confounding factors:

> Impact of Label Distribution

```sh
# balanced distribution
npm run dataset:balanced
npm run benchmark

# unbalanced distribution
npm run dataset:unbalanced
npm run benchmark
```

> Impact of function words

```sh
npm run dataset:balanced
npm run benchmark
```

> Impact of Language Consistency in Issue Tickets

```sh
# baseline
npm run dataset:english:baseline
npm run benchmark

# english
npm run dataset:english
npm run benchmark
```

> Presence of Code Snippets in Issue Tickets

```sh
# baseline
npm run dataset:nosnip:baseline
npm run benchmark

# no snippets
npm run dataset:nosnip
npm run benchmark
```

#### generate dataset:

Datasets can be downloaded either using `npm run dataset:balanced` or `npm run dataset:unbalanced`.
The datasets were generated using github archive's which can be accessed through google [BigQuery](https://bigquery.cloud.google.com).

Add the query below to your BigQuery console and adjust if needed (e.g., add `__label__` prefix to labels, truncate issues to create a balanced dataset, etc.).

```sql
-- unbalanced dataset

SELECT
  label,
  CONCAT(title, ' ', REGEXP_REPLACE(body, '(\r|\n|\r\n)',' ')) as text
FROM (
  SELECT
    LOWER(JSON_EXTRACT_SCALAR(payload, '$.issue.labels[0].name')) AS label,
    JSON_EXTRACT_SCALAR(payload, '$.issue.title') AS title,
    JSON_EXTRACT_SCALAR(payload, '$.issue.body') AS body
  FROM
    `githubarchive.day.201802*`
  WHERE
    _TABLE_SUFFIX BETWEEN '01' AND '10'
    AND type = 'IssuesEvent'
    AND JSON_EXTRACT_SCALAR(payload, '$.action') = 'closed' )
WHERE 
  (label = 'bug' OR label = 'enhancement' OR label = 'question')
  AND body != 'null';
```

#### run serverless app:

You need a `.env` file in order to run the github app.
The file should look like this:

```
GITHUB_CERT="<private key>"
GITHUB_SECRET=123456
GITHUB_APP_ID=123
PORT=3000
```

Note: When running app in production, environment variables should be provided by host.

#### references:

- [Building GitHub Apps](https://developer.github.com/apps/building-github-apps/)
- [Fasttext](https://fasttext.cc)
