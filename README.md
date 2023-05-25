# 完成ファイル
abema_db_table_modify.mdの方を確認してください。

# ER図確認手順
vscodeにて
* Markdown Preview Mermaid Supportを拡張機能追加
* `ctrl+shit+v`でプレビュー表示

# テーブル構築手順
* 動作環境
macOS Monterey 12.6
VScode 1.78.0
DB: PostgreSQL

1. 'brew search postgresql'でインストールできるバージョンを確認
2. 'brew postgresql@バージョン'でインストールできる新しいものを指定
3. `psql --version`でバージョン確認
4. `brew services start postgresql'でPostgreSQLを起動する
5. `psql -h localhost -p 5432 -U OSのユーザー名 -d postgres`でPostgreSQLに接続する
6. 接続後、`CREATE DATABASE db;`でDB作成
7. 以下のSQL文で構築
```
CREATE TABLE channels (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);

CREATE TABLE slots (
  id SERIAL PRIMARY KEY,
  broadcast_start_time TIMESTAMP NOT NULL,
  broadcast_finish_time TIMESTAMP NOT NULL,
  channel_id INT NOT NULL REFERENCES channels
);

CREATE TABLE series_programs (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  detail TEXT
);

CREATE TABLE series_seasons (
  id SERIAL PRIMARY KEY,
  number INT NULL,
  series_program_id INT NOT NULL REFERENCES series_programs
);

CREATE TABLE series_episodes (
  id SERIAL PRIMARY KEY,
  season_id INT NOT NULL REFERENCES series_seasons,
  number INT NOT NULL,
  title VARCHAR(255) NOT NULL UNIQUE,
  detail TEXT UNIQUE,
  video_time TIME NOT NULL,
  release_date DATE NOT NULL
);

CREATE TABLE single_programs (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  detail TEXT UNIQUE,
  video_time TIME NOT NULL,
  release_date DATE NOT NULL
);

CREATE TABLE viewcounts (
  id SERIAL PRIMARY KEY,
  episode_id INT NULL REFERENCES series_episodes,
  program_id INT NULL REFERENCES single_programs,
  slot_id INT NOT NULL REFERENCES slots,
  count INT NOT NULL
);

CREATE TABLE categories (
   id SERIAL PRIMARY KEY, 
   name VARCHAR(255) NOT NULL UNIQUE 
);

CREATE TABLE series_program_categories (
   program_id INT NOT NULL REFERENCES series_programs, 
   category_id INT NOT NULL REFERENCES categories, 
   PRIMARY KEY (program_id, category_id) 
);

CREATE TABLE single_program_categories ( 
   program_id INT NOT NULL REFERENCES single_programs, 
   category_id INT NOT NULL REFERENCES categories, 
   PRIMARY KEY (program_id, category_id) 
);

```
8. 以下のSQL文でサンプルデータ挿入
```
-- channels テーブルにデータを挿入
INSERT INTO channels (name) VALUES
('NHK'),
('TBS'),
('フジテレビ');

-- slots テーブルにデータを挿入
INSERT INTO slots (broadcast_start_time, broadcast_finish_time, channel_id) VALUES
('09:00:00', '10:00:00', 1), -- NHKで午前9時から午前10時まで
('21:00:00', '22:00:00', 1), -- NHKで午後9時から午後10時まで
('19:00:00', '20:54:00', 2), -- TBSで午後7時から午後8時54分まで
('21:00:00', '23:24:00', 2), -- TBSで午後9時から午後11時24分まで
('18:30:00', '19:57:00', 3), -- フジテレビで午後6時30分から午後7時57分まで
('21:00:00', '23:10:00', 3); -- フジテレビで午後9時から午後11時10分まで

-- series_programs テーブルにデータを挿入
INSERT INTO series_programs (name, detail) VALUES
('半沢直樹', '銀行員が不正や圧力に立ち向かう人気ドラマ'),
('鬼滅の刃', '鬼と戦う少年たちの冒険物語');

-- series_seasons テーブルにデータを挿入
INSERT INTO series_seasons (number, series_program_id) VALUES
(1, 1), -- 半沢直樹 第1シーズン
(2, 1), -- 半沢直樹 第2シーズン
(1, 2); -- 鬼滅の刃 第1シーズン

-- series_episodes テーブルにデータを挿入
INSERT INTO series_episodes (season_id, number, title, detail, video_time, release_date) VALUES
(1, 1, '倍返し編', '半沢直樹は大和田常務から不正な融資を強要されるが...', '01:00:00', '2020-07-19'), -- 半沢直樹 第1シーズン 第1話
(1, 2, '300億円編', '半沢直樹は東京中央銀行本店へ異動するが...', '01:00:00', '2020-07-26'), -- 半沢直樹 第1シーズン 第2話
(2, 1, '舞台裏編', '半沢直樹は大阪スティールへ出向するが...', '01:15:00', '2020-09-13'), -- 半沢直樹 第2シーズン 第1話
(2, 2, '決戦編', '半沢直樹は大阪スティールの不正を暴くが...', '01:15:00', '2020-09-20'), -- 半沢直樹 第2シーズン 第2話
(3, 1, '鬼殺隊入隊編', '竈門炭治郎は家族を鬼に殺されるが...', '00:24:00', '2019-04-06'), -- 鬼滅の刃 第1シーズン 第1話
(3, 2, '鬼舞辻無惨編', '竈門炭治郎は鬼舞辻無惨と対峙するが...', '01:57:00', '2020-10-16'); -- 鬼滅の刃 第1シーズン 第2話

-- single_programs テーブルにデータを挿入
INSERT INTO single_programs (name, detail, video_time, release_date) VALUES
('24時間テレビ', '毎年夏に放送されるチャリティー番組', '24:00:00', '2020-08-22'),
('紅白歌合戦', '毎年大晦日に放送される音楽番組', '04:30:00', '2020-12-31');

-- viewcounts テーブルにデータを挿入
INSERT INTO viewcounts (episode_id, program_id, slot_id, count) VALUES
(1, NULL, 2, 2000000), -- 半沢直樹 第1シーズン 第1話がNHKで午後9時から午後10時まで放送された場合
(2, NULL, 2, 1800000), -- 半沢直樹 第1シーズン 第2話がNHKで午後9時から午後10時まで放送された場合
(3, NULL, 4, 1500000), -- 半沢直樹 第2シーズン 第1話がTBSで午後9時から午後11時24分まで放送された場合
(4, NULL, 4, 1700000), -- 半沢直樹 第2シーズン 第2話がTBSで午後9時から午後11時24分まで放送された場合
(5, NULL, 6, 1000000), -- 鬼滅の刃 第1シーズン 第1話がフジテレビで午後6時30分から午後7時57分まで放送された場合
(6, NULL, 6, 2000000), -- 鬼滅の刃 第1シーズン 第2話がフジテレビで午後9時から午後11時10分まで放送された場合
(NULL, 1, 3, 500000), -- 24時間テレビがTBSで午後7時から午後8時54分まで放送された場合
(NULL, 2, 2, 3000000); -- 紅白歌合戦がNHKで午後9時から午後10時まで放送された場合

-- categories テーブルにデータを挿入
INSERT INTO categories (name) VALUES
('ドラマ'),
('アニメ'),
('バラエティ');

-- series_program_categories テーブルにデータを挿入
INSERT INTO series_program_categories (program_id, category_id) VALUES
(1, 1), -- 半沢直樹はドラマのカテゴリーに属する
(2, 2); -- 鬼滅の刃はアニメのカテゴリーに属する

-- single_program_categories テーブルにデータを挿入
INSERT INTO single_program_categories (program_id, category_id) VALUES
(1, 3), -- 24時間テレビはバラエティのカテゴリーに属する
(2, 3); -- 紅白歌合戦はバラエティのカテゴリーに属する

```

# 気づき
## インデックスを入れる基準
* 更新頻度が低いカラム：更新頻度が低いカラムは、インデックスのメンテナンスコストが低いため、インデックスを入れても問題ない。逆に更新頻度が高いカラムは、インデックスのメンテナンスコストが高いため、インデックスを入れるとパフォーマンスに影響する可能性がある。
* 検索頻度が高いカラム：検索頻度が高いカラムは、インデックスを入れることで検索速度を向上させることができる。逆に検索頻度が低いカラムは、インデックスを入れてもあまり効果がないため、インデックスを入れなくても良い。
* カーディナリティが高いカラム：カーディナリティが高いカラムは、値の重複が少ないため、インデックスを入れることで検索効率が高まる。逆にカーディナリティが低いカラムは、値の重複が多いため、インデックスを入れてもあまり効果がない可能性。
## その他
* パフォーマンスとテーブルの分割のバランス感が難しい
