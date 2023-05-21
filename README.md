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
CREATE TABLE channel (
  channel_id SERIAL PRIMARY KEY,
  channel_name VARCHAR(255) NOT NULL
);

CREATE TABLE slot (
  slot_id SERIAL PRIMARY KEY,
  broadcast_start_time TIMESTAMP NOT NULL,
  broadcast_finish_time TIMESTAMP NOT NULL,
  channel_id INT NOT NULL REFERENCES channel
);

CREATE TABLE series_program (
  series_program_id SERIAL PRIMARY KEY,
  series_program_name VARCHAR(255) NOT NULL UNIQUE,
  series_program_detail TEXT
);

CREATE TABLE series_season (
  series_season_id SERIAL PRIMARY KEY,
  series_season_number INT NULL,
  series_program_id INT NOT NULL REFERENCES series_program
);

CREATE TABLE series_episode (
  series_episode_id SERIAL PRIMARY KEY,
  series_season_id INT NOT NULL REFERENCES series_season,
  series_episode_number INT NOT NULL,
  series_episode_title VARCHAR(255) NOT NULL UNIQUE,
  series_episode_detail TEXT UNIQUE,
  video_time TIME NOT NULL,
  release_date DATE NOT NULL
);

CREATE TABLE single_program (
  single_program_id SERIAL PRIMARY KEY,
  single_program_name VARCHAR(255) NOT NULL UNIQUE,
  single_program_detail TEXT UNIQUE,
  video_time TIME NOT NULL,
  release_date DATE NOT NULL
);

CREATE TABLE viewcount (
  view_count_id SERIAL PRIMARY KEY,
  series_episode_id INT NULL REFERENCES series_episode,
  single_program_id INT NULL REFERENCES single_program,
  slot_id INT NOT NULL REFERENCES slot,
  view_count INT NOT NULL
);

CREATE TABLE category (
  category_id SERIAL PRIMARY KEY,
  category_name VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE series_program_category (
  series_program_id INT NOT NULL REFERENCES series_program,
  category_id INT NOT NULL REFERENCES category,
  PRIMARY KEY (series_program_id, category_id)
);

CREATE TABLE single_program_category (
  single_program_id INT NOT NULL REFERENCES single_program,
  category_id INT NOT NULL REFERENCES category,
  PRIMARY KEY (single_program_id, category_id)
);

```
8. 以下のSQL文でサンプルデータ挿入
```
-- channelテーブルにサンプルデータを挿入
INSERT INTO channel (channel_name) VALUES
('NHK'),
('TBS'),
('Fuji TV'),
('TV Asahi'),
('TV Tokyo');

-- categoryテーブルにサンプルデータを挿入
INSERT INTO category (category_name) VALUES
('News'),
('Drama'),
('Comedy'),
('Anime'),
('Documentary');

-- series_programテーブルにサンプルデータを挿入
INSERT INTO series_program (series_program_name, series_program_detail) VALUES
('Hanzawa Naoki', 'A bank employee who fights against corruption and injustice.'),
('Dragon Ball', 'A martial arts adventure of a boy named Goku and his friends.'),
('Crayon Shin-chan', 'A comedy about the daily life of a mischievous boy and his family.'),
('Detective Conan', 'A mystery about a high school detective who is turned into a child and solves various cases.'),
('Planet Earth', 'A documentary about the natural wonders and wildlife of the Earth.');

-- series_seasonテーブルにサンプルデータを挿入
INSERT INTO series_season (series_season_number, series_program_id) VALUES
(1, 1),
(2, 1),
(1, 2),
(2, 2),
(3, 2),
(4, 2),
(5, 2),
(NULL, 3),
(NULL, 4),
(1, 5),
(2, 5);

-- series_episodeテーブルにサンプルデータを挿入
INSERT INTO series_episode (series_season_id, series_episode_number, series_episode_title, series_episode_detail, video_time, release_date) VALUES
(1, 1, 'Unjustified Resignation', 'Hanzawa is forced to resign after his client goes bankrupt.', '01:00:00', '2020-07-05'),
(1, 2, '300% Payback', 'Hanzawa tries to recover the loan from the bankrupt company.', '01:00:00', '2020-07-12'),
(1, 3, 'The Trap of the Conglomerate', 'Hanzawa faces a new challenge from a powerful rival.', '01:00:00', '2020-07-19'),
(1, 4, 'The Secret of the Missing 500 Million Yen', 'Hanzawa uncovers a hidden fraud involving his boss.', '01:00:00', '2020-07-26'),
(1, 5, 'The Showdown with Teikoku Bank', 'Hanzawa confronts the president of Teikoku Bank.', '01:00:00', '2020-08-02'),
(1, 6, 'The Miracle of Osaka', 'Hanzawa achieves a stunning victory over Teikoku Bank.', '01:00:00', '2020-08-09'),
(1, 7, 'The Tokyo Prosecutor''s Office Moves Out', 'Hanzawa is investigated by the prosecutors for his actions.', '01:00:00', '2020-08-16'),
(1, 8, 'Don''t Let Go of Justice', 'Hanzawa fights back against the prosecutors with the help of his allies.', '01:00:00', '2020-08-23'),
(1, 9, 'The Truth Behind the Financial Crimes', 'Hanzawa exposes the mastermind behind the financial crimes.', '01:00:00', '2020-08-30'),
(1, 10, 'The Last Fight', 'Hanzawa faces his final enemy in a climactic showdown.', '01:30:00', '2020-09-06'),
(2, 1, 'The Unforgivable One', 'Hanzawa is transferred to Tokyo Central Securities and meets a new enemy.', '01:00:00', '2020-09-13'),
(2, 2, 'The Fallen Eagle', 'Hanzawa tries to save a struggling company from a hostile takeover.', 
'01:00:00', 
'2020-09-20');
-- 略

-- single_programテーブルにサンプルデータを挿入
INSERT INTO single_program (single_program_name, single_program_detail, video_time,
release_date) VALUES
('Spirited Away', 
'A fantasy about a girl who enters a magical world and tries to save her parents.',
'02:05:00',
'2001-07-20'),
('Your Name',
'A romance about two teenagers who switch bodies and fall in love.',
'01:46:00',
'2016-08-26'),
('The King''s Speech',
'A drama about King George VI who overcomes his stammer with the help of a speech therapist.',
'01:58:00',
'2010-09-06'),
('Joker',
'A thriller about the origin story of the iconic Batman villain.',
'02:02:00',
'2019-08-31'),
('Parasite',
'A dark comedy about a poor family who infiltrates a wealthy household.',
'02:12:00',
'2019-05-21');

-- slotテーブルにサンプルデータを挿入
INSERT INTO slot (broadcast_start_time,
broadcast_finish_time,
channel_id) VALUES
('2020-09-20 21:00:00',
'2020-09-20 22:00:00',
2),
('2020-09-20 22:30:00',
'2020-09-21 01:15:00',
3),
('2020-09-21 19:30:00',
'2020-09-21 20:54:00',
4),
('2020-09-21 21:15:00',
'2020-09-21 23:17:00',
5),
('2020-09-22 18:25:00',
'2020-09-22 20:48:00',
3);
-- 略

-- viewcountテーブルにサンプルデータを挿入
INSERT INTO viewcount (series_episode_id,
single_program_id,
slot_id,
view_count) VALUES
(11,
NULL,
1,
1500000),
(NULL,
2,
2,
1200000),
(NULL,
5,
3,
800000),
(NULL,
4,
4,
1000000),
(NULL,
3,
5,
900000);
-- 略

-- series_program_categoryテーブルにサンプルデータを挿入
INSERT INTO series_program_category (series_program_id,
category_id) VALUES
(1,
2),
(2,
4),
(3,
3),
(4,
2),
(5,
5);

-- single_program_categoryテーブルにサンプルデータを挿入
INSERT INTO single_program_category (single_program_id,
category_id) VALUES
(1,
4),
(2,
4),
(3,
2),
(4,
3),
(5,
3);
```

# 気づき
## インデックスを入れる基準
* 更新頻度が低いカラム：更新頻度が低いカラムは、インデックスのメンテナンスコストが低いため、インデックスを入れても問題ない。逆に更新頻度が高いカラムは、インデックスのメンテナンスコストが高いため、インデックスを入れるとパフォーマンスに影響する可能性がある。
* 検索頻度が高いカラム：検索頻度が高いカラムは、インデックスを入れることで検索速度を向上させることができる。逆に検索頻度が低いカラムは、インデックスを入れてもあまり効果がないため、インデックスを入れなくても良い。
* カーディナリティが高いカラム：カーディナリティが高いカラムは、値の重複が少ないため、インデックスを入れることで検索効率が高まる。逆にカーディナリティが低いカラムは、値の重複が多いため、インデックスを入れてもあまり効果がない可能性。
## その他
* パフォーマンスとテーブルの分割のバランス感が難しい
