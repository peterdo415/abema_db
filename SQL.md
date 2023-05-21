1. エピソード視聴数トップ3のエピソードタイトルと視聴数
```
-- エピソードタイトルと視聴数を結合する
SELECT se.series_episode_title, vc.view_count
FROM series_episode se
JOIN viewcount vc
ON se.series_episode_id = vc.series_episode_id;

-- 視聴数で降順に並べ替えて、上位3件を取得する
SELECT se.series_episode_title, vc.view_count
FROM series_episode se
JOIN viewcount vc
ON se.series_episode_id = vc.series_episode_id
ORDER BY vc.view_count DESC
LIMIT 3;
```
2. 