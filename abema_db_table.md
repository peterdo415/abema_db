# 記録用

```mermaid

erDiagram
 CHANNELS ||--|{ SLOTS : ""
 SLOTS ||--o{ EPISODES : ""
 PROGRAMS ||--|{ SEASONS : ""
 SEASONS ||--o{ EPISODES : ""
 PROGRAMS ||--|{ PROGRAMCATEGORIES : ""
 CATEGORYS ||--|{ PROGRAMCATEGORIES : ""
 SLOTS ||--|| VIEWCOUNTS : ""
 EPISODES ||--|| VIEWCOUNTS : ""

 CHANNELS {
 channel_id int "primary"
 channel_name string 
 }

 SLOTS {
 slot_id int "primary"
 broadcast_start_time datetime
 broadcast_finish_time datetime
 channel_id int "foreign"
 }
 PROGRAMS {
 program_id int "primary"
 program_name string
 program_detail text
 }
 SEASONS {
 season_id int "primary"
 season_number int
 program_id int "foreign"
 }
 EPISODES {
 episode_id int "primary"
 episode_number int "NULL"
 episode_title string "NULL"
 episode_detail text "NULL"
 video_time time
 release_date date
 season_id int "foreign NULL"
 }
 VIEWCOUNTS {
 view_count_id int "primary"
 episode_id int "foreign"
 slot_id int "foreign"
 view_count int
 }
 CATEGORIES {
 category_id int "primary"
 category_name string
 }
 
 PROGRAMCATEGORIES {
 program_id int "primary foreign"
 category_id int "primary foreign"
 }

```

# メモ
* チャンネルは複数
* 各チャンネルは時間帯毎の番組枠を持つ
* 番組枠1つに対して1つの番組が入る
* エピソードは最小単位としての番組、そのため番組枠が持つのはエピソード
* エピソードはシーズン数、エピソード数、タイトル、エピソード詳細、動画時間、公開日、視聴数をもつ
* エピソードは自分が所属するシーズンを参照する
* `series_id'と'program_id'はどちらかしか持たない
