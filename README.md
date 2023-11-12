## ステップ1/テーブル設計

テーブル: チャンネル/channels
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|TINYINT(4)UNSIGNED||PRIMARY||YES|
|name|VARCHAR(255)||UNIQUE|||

テーブル: 時間帯の番組枠/time_slots
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|BIGINT(20)UNSIGNED||PRIMARY||YES|
|channel_id|TINYINT(4)UNSIGNED||FOREIGN|||
|episode_id|SMALLINT(5)UNSIGNED||FOREIGN|||
|strat_at|DATETIME|||||
|end_at|DATETIME|||||
|num_of_views|BIGINT(20)UNSIGNED|||0||

テーブル: 番組/programs
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|BIGINT(20)UNSIGNED||PRIMARY||YES|
|program_title|VARCHAR(255)||INDEX|||
|program_desc|VARCHAR(1024)|||||

中間テーブル: 番組_ジャンル/program_genres
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|BIGINT(20)UNSIGNED||PRIMARY||YES|
|program_id|BIGINT(20)UNSIGNED||FOREIGN|||
|genre_id|SMALLINT(4)UNSIGNED||FOREIGN|||

テーブル: ジャンル/genres
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|SMALLINT(4)UNSIGNED||PRIMARY||YES|
|name|VARCHAR(255)||UNIQUE|||

テーブル: エピソード/episodes
|カラム名|データ型|NULL|キー|初期値|AUTO INCREMENT|
|-|-|-|-|-|-|
|id|BIGINT(20)UNSIGNED||PRIMARY||YES|
|program_id|BIGINT(20)UNSIGNED||FOREIGN|||
|season|SMALLINT(6)UNSIGNED|YES||||
|episode_num|SMALLINT(6)UNSIGNED|||||
|episode_title|VARCHAR(255)||INDEX|||
|episode_desc|VARCHAR(1024)|||||
|duration|TIME|||||
|release_on|DATE|||||

## ステップ2/テーブル構築 サンプルデータ入力

### データベース作成

```sql
CREATE DATABASE internet_tv;
USE internet_tv;
```

## テーブル作成

```sql
CREATE TABLE channels (
    id   tinyint(4)   UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name varchar(255)          NOT NULL
);

CREATE TABLE programs (
    id            BIGINT(20)    UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    program_title VARCHAR(255)           NOT NULL,
    program_desc  VARCHAR(1024) UNIQUE   NOT NULL,
                  INDEX p_title (program_title)
);

CREATE TABLE genres (
    id   SMALLINT(4)  UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) UNIQUE   NOT NULL
);

CREATE TABLE program_genres (
    id         BIGINT(20)  UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    program_id BIGINT(20)  UNSIGNED NOT NULL,
    genre_id   SMALLINT(4) UNSIGNED NOT NULL,
               FOREIGN KEY (program_id) REFERENCES programs(id),
               FOREIGN KEY (genre_id) REFERENCES genres(id)
);

CREATE TABLE episodes (
    id            BIGINT(20)    UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    program_id    BIGINT(20)    UNSIGNED NOT NULL,
    season        SMALLINT(6)   UNSIGNED,
    episode_num   SMALLINT(6)   UNSIGNED NOT NULL,
    episode_title VARCHAR(255)           NOT NULL,
    episode_desc  VARCHAR(1024)          NOT NULL,
    duration      TIME                   NOT NULL,
    release_on    DATE                   NOT NULL,
                  INDEX e_title (episode_title)
);

CREATE TABLE time_slots (
    id           BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    channel_id   TINYINT(4) UNSIGNED NOT NULL,
    episode_id   BIGINT(20) UNSIGNED NOT NULL,
    start_at     DATETIME            NOT NULL,
    end_at       DATETIME            NOT NULL,
    num_of_views BIGINT(20) UNSIGNED NOT NULL  DEFAULT 0,
                 FOREIGN KEY (channel_id) REFERENCES channels(id),
                 FOREIGN KEY (episode_id) REFERENCES episodes(id)
);
```

### サンプルデータ

[サンプルデータ](sampledata.sql)

## ステップ3/クエリ

### 1.よく見られているエピソードを知りたいです。エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください

```sql
SELECT e.episode_title AS 'エピソードタイトル',
       SUM(t.num_of_views) AS '視聴数'
  FROM time_slots AS t
       INNER JOIN episodes AS e
       ON t.episode_id = e.id
 GROUP BY t.episode_id
 ORDER BY SUM(t.num_of_views) DESC
 LIMIT 3
;
```

出力
```
+-----------------------------------+---------+
| エピソードタイトル                | 視聴数  |
+-----------------------------------+---------+
| 推理ドラマ/エピソード4のタイトル  | 1499340 |
| 推理ドラマ/エピソード10のタイトル | 1293086 |
| 推理ドラマ/エピソード5のタイトル  | 1215966 |
+-----------------------------------+---------+
3 rows in set (0.001 sec)
```

### 2.よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください

```sql
SELECT p.program_title AS '番組タイトル',
       e.season AS 'シーズン数',
       e.episode_num AS 'エピソード数',
       e.episode_title AS 'エピソードタイトル',
       SUM(t.num_of_views) AS '視聴数'
  FROM time_slots AS t
       INNER JOIN episodes AS e
       ON t.episode_id = e.id
       INNER JOIN programs AS p
       ON e.program_id = p.id
 GROUP BY t.episode_id
 ORDER BY SUM(t.num_of_views) DESC
 LIMIT 3
;
```

出力

```
+----------------------+------------+--------------+-----------------------------------+---------+
| 番組タイトル         | シーズン数 | エピソード数 | エピソードタイトル                | 視聴数  |
+----------------------+------------+--------------+-----------------------------------+---------+
| 推理ドラマのタイトル |          1 |            4 | 推理ドラマ/エピソード4のタイトル  | 1499340 |
| 推理ドラマのタイトル |          1 |           10 | 推理ドラマ/エピソード10のタイトル | 1293086 |
| 推理ドラマのタイトル |          1 |            5 | 推理ドラマ/エピソード5のタイトル  | 1215966 |
+----------------------+------------+--------------+-----------------------------------+---------+
3 rows in set (0.001 sec)
```

### 3.本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします

```sql
SELECT c.name AS 'チャンネル名',
       t.start_at AS '放送開始時刻',
       t.end_at AS '放送終了時刻',
       e.season AS 'シーズン数',
       e.episode_num AS 'エピソード数',
       e.episode_title AS 'エピソードタイトル',
       e.episode_desc AS 'エピソード詳細'
  FROM time_slots AS t
       INNER JOIN episodes AS e
       ON t.episode_id = e.id
       INNER JOIN channels AS c
       ON t.channel_id = c.id
 WHERE t.start_at LIKE '2023-11-01%'
;
```

出力

```
+--------------+---------------------+---------------------+------------+--------------+------------------------------------------+--------------------------------------+
| チャンネル名 | 放送開始時刻        | 放送終了時刻        | シーズン数 | エピソード数 | エピソードタイトル                       | エピソード詳細                       |
+--------------+---------------------+---------------------+------------+--------------+------------------------------------------+--------------------------------------+
| ドラマ1      | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | 推理ドラマ/エピソード1のタイト ル         | 推理ドラマ/エピソード1の詳細         |
| ドラマ2      | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | 恋愛ドラマ/エピソード1のタイト ル         | 恋愛ドラマ/エピソード1の詳細         |
| アニメ1      | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | ファンタジーアニメ/エピソード1 のタイトル | ファンタジーアニメ/エピソード1の詳細 |
| アニメ2      | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | アクションアニメ/エピソード1の タイトル   | アクションアニメ/エピソード1の詳細   |
| スポーツ     | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | サッカー番組/エピソード1のタイ トル       | サッカー番組/エピソード1の詳細       |
| ペット       | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | ペット番組/エピソード1のタイト ル         | ペット番組/エピソード1の詳細         |
| ニュース     | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | ニュース番組/エピソード1のタイ トル       | ニュース番組/エピソード1の詳細       |
| 音楽         | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | 音楽番組/エピソード1のタイトル           | 音楽番組/エピソード1の詳細           |
| 映画         | 2023-11-01 00:00:00 | 2023-11-01 02:00:00 |       NULL |            1 | アクション映画のタイトル                 | アクション映画の詳細                 |
| 将棋         | 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | 将棋番組/エピソード1のタイトル           | 将棋番組/エピソード1の詳細           |
+--------------+---------------------+---------------------+------------+--------------+------------------------------------------+--------------------------------------+
10 rows in set (0.000 sec)
```

### 4.ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください

```sql
-- NOW()を11/01と仮定、本日から一週間(11/04~11/10)のデータを取得
SELECT t.start_at AS '放送開始時刻',
       t.end_at AS '放送終了時刻',
       e.season AS 'シーズン数',
       e.episode_num AS 'エピソード数',
       e.episode_title AS 'エピソードタイトル',
       e.episode_desc AS 'エピソード詳細'
  FROM time_slots AS t
       INNER JOIN episodes AS e
       ON t.episode_id = e.id
       INNER JOIN channels AS c
       ON t.channel_id = c.id
 WHERE t.channel_id = 1
   AND t.start_at < ('2023-11-01 00:00:00' + INTERVAL 1 WEEK)
 ORDER BY start_at ASC
;
```

出力

```
+---------------------+---------------------+------------+--------------+----------------------------------+------------------------------+
| 放送開始時刻        | 放送終了時刻        | シーズン数 | エピソード数 | エピソードタイトル               | エピソード 詳細               |
+---------------------+---------------------+------------+--------------+----------------------------------+------------------------------+
| 2023-11-01 00:00:00 | 2023-11-01 00:30:00 |          1 |            1 | 推理ドラマ/エピソード1のタイトル | 推理ドラマ/エピソード1の詳細 |
| 2023-11-02 00:00:00 | 2023-11-02 00:30:00 |          1 |            1 | 推理ドラマ/エピソード1のタイトル | 推理ドラマ/エピソード1の詳細 |
| 2023-11-02 02:00:00 | 2023-11-02 02:30:00 |          1 |            2 | 推理ドラマ/エピソード2のタイトル | 推理ドラマ/エピソード2の詳細 |
| 2023-11-03 02:00:00 | 2023-11-03 02:30:00 |          1 |            2 | 推理ドラマ/エピソード2のタイトル | 推理ドラマ/エピソード2の詳細 |
| 2023-11-03 05:00:00 | 2023-11-03 05:30:00 |          1 |            3 | 推理ドラマ/エピソード3のタイトル | 推理ドラマ/エピソード3の詳細 |
| 2023-11-04 05:00:00 | 2023-11-04 05:30:00 |          1 |            3 | 推理ドラマ/エピソード3のタイトル | 推理ドラマ/エピソード3の詳細 |
| 2023-11-04 07:00:00 | 2023-11-04 07:30:00 |          1 |            4 | 推理ドラマ/エピソード4のタイトル | 推理ドラマ/エピソード4の詳細 |
| 2023-11-05 07:00:00 | 2023-11-05 07:30:00 |          1 |            4 | 推理ドラマ/エピソード4のタイトル | 推理ドラマ/エピソード4の詳細 |
| 2023-11-05 08:00:00 | 2023-11-05 08:30:00 |          1 |            5 | 推理ドラマ/エピソード5のタイトル | 推理ドラマ/エピソード5の詳細 |
| 2023-11-06 08:00:00 | 2023-11-06 08:30:00 |          1 |            5 | 推理ドラマ/エピソード5のタイトル | 推理ドラマ/エピソード5の詳細 |
| 2023-11-06 12:00:00 | 2023-11-06 12:30:00 |          1 |            6 | 推理ドラマ/エピソード6のタイトル | 推理ドラマ/エピソード6の詳細 |
| 2023-11-07 12:00:00 | 2023-11-07 12:30:00 |          1 |            6 | 推理ドラマ/エピソード6のタイトル | 推理ドラマ/エピソード6の詳細 |
| 2023-11-07 16:00:00 | 2023-11-07 16:30:00 |          1 |            7 | 推理ドラマ/エピソード7のタイトル | 推理ドラマ/エピソード7の詳細 |
+---------------------+---------------------+------------+--------------+----------------------------------+------------------------------+
13 rows in set (0.000 sec)
```

### 5.(advanced) 直近一週間で最も見られた番組が知りたいです。直近一週間に放送された番組の中で、エピソード視聴数合計トップ2の番組に対して、番組タイトル、視聴数を取得してください

```sql
-- NOW()を11/10と仮定、直近一週間(11/04~11/10)で最も見られた2番組を取得
SELECT pttn.pt AS '番組タイトル',
       SUM(pttn.tn) AS '番組ごとの視聴数'
  FROM (SELECT p.program_title AS pt,
	       SUM(t.num_of_views) AS tn
	  FROM time_slots AS t
	       INNER JOIN episodes AS e
               ON t.episode_id = e.id
               INNER JOIN programs AS p
               ON e.program_id = p.id
	 WHERE t.start_at > ('2023-11-10 00:00:00' - INTERVAL 1 WEEK)
	 GROUP BY t.episode_id) AS pttn
 GROUP BY pttn.pt
 ORDER BY SUM(pttn.tn) DESC
 LIMIT 2
;
```

出力

```
+------------------------+------------------+
| 番組タイトル           | 番組ごとの視聴数 |
+------------------------+------------------+
| 推理ドラマのタイトル   |          8286319 |
| サッカー番組のタイトル |          5277940 |
+------------------------+------------------+
2 rows in set (0.001 sec)
```

### 6.(advanced) ジャンルごとの番組の視聴数ランキングを知りたいです。番組の視聴数ランキングはエピソードの平均視聴数ランキングとします。ジャンルごとに視聴数トップの番組に対して、ジャンル名、番組タイトル、エピソード平均視聴数を取得してください。

```sql
-- 番組の視聴数ランキング(エピソードの平均視聴数ランキング)を取得
WITH avg_program AS (
    SELECT p.id AS pi, p.program_title AS pt,
           AVG(t.num_of_views) AS atn
      FROM time_slots AS t
           INNER JOIN episodes AS e
           ON t.episode_id = e.id
           INNER JOIN programs AS p
           ON e.program_id = p.id
     GROUP BY p.id
)

-- ジャンルごとの視聴数トップを取得
SELECT g.name AS "ジャンル名", pt AS "番組タイトル",
       atn AS "エピソード平均視聴数"
  FROM avg_program AS ap
       INNER JOIN program_genres AS pg
       ON ap.pi = pg.program_id
       INNER JOIN genres AS g
       ON pg.genre_id = g.id
 WHERE atn = (SELECT MAX(atn)
                FROM avg_program AS ap2
                     INNER JOIN program_genres AS pg2
                     ON ap2.pi = pg2.program_id
		     INNER JOIN genres AS g2
                     ON pg2.genre_id = g2.id
               WHERE g.name = g2.name)
;
```

出力

```
+------------------+--------------------------------+----------------------+
| ジャンル名       | 番組タイトル                   | エピソード平均視聴数 |
+------------------+--------------------------------+----------------------+
| アクション       | アクションアニメのタイトル     |          410479.4000 |
| アニメ           | ファンタジーアニメのタイトル   |          447102.1000 |
| サスペンス       | サスペンス映画のタイトル       |          313763.0000 |
| サッカー         | サッカー番組のタイトル         |          643092.3000 |
| スポーツ         | サッカー番組のタイトル         |          643092.3000 |
| ドキュメンタリー | ドキュメンタリー映画のタイトル |          419587.0000 |
| ドラマ           | 推理ドラマのタイトル           |          485087.4000 |
| ニュース         | ニュース番組のタイトル         |          568824.4000 |
| ファンタジー     | ファンタジーアニメのタイトル   |          447102.1000 |
| ペット           | ペット番組のタイトル           |          594592.6000 |
| ミュージカル     | ミュージカル映画のタイトル     |          289964.0000 |
| 動物             | ペット番組のタイトル           |          594592.6000 |
| 将棋             | 将棋番組のタイトル             |          484537.0000 |
| 恋愛             | 恋愛映画のタイトル             |          535301.0000 |
| 推理             | 推理ドラマのタイトル           |          485087.4000 |
| 映画             | 恋愛映画のタイトル             |          535301.0000 |
| 音楽             | 音楽番組のタイトル             |          383013.1000 |
+------------------+--------------------------------+----------------------+
17 rows in set (0.008 sec)
```
