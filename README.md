# D&A Project

This database is loosely based on a databse a streaming service like YouTube would use.

Instructions to use:
1. run in MySQL 
   ```
   create database daproj
   use daproj```
2. run the sql dump provided, `daproj.sql`
3. you can now use the database

## 1. Upload a video
We ask the user to input all the information we require to insert a video into our sql database.

```python
    Q1 = f"""INSERT INTO video (title, duration, upload_time, videoId)
            VALUES( '{inp['title']}', {inp['duration']}, '{inp['upload_time']}', {vidID} )"""

    Q2 = f"INSERT INTO upload VALUES ({accID}, {vidID})"
```

## 2. Change the title of a video
We ask the user for the `videoID` of the video whose title they want to change. We then ask the user for the new title and check if its valid.
If it is valid, we update the database.

```python
	Q = f"""SELECT v.title as title
			FROM video v
			WHERE v.videoID = {vidID}"""
```

## 3. Delete a comment
We ask the user for the `accountID` whose comment they want to delete. Next, we ask them for a keyword from the comment to be deleted.
An indexed list of all comments made by that account, containing that keyword is displayed. The user is asked to enter which comment they would like to delete.
If they enter a valid index, the comment is deleted.

```python
    Q = f"""SELECT comment.commentID as cID, comment.content as c
        FROM comment, comments_on co
        WHERE comment.commentID = co.commentID
        AND comment.content LIKE '%{keyword}%'
        AND co.accountID = {accountID}"""

	Q = f"""DELETE
		FROM comment
		WHERE commentID = {del_cID}"""
```

## 4. List all video uploaded by an account
The user is asked for the accountID of the account whose videos they want a list of.
A list of all videos, with some extra information, uploaded by that account is then displayed.

```python
    Q = f"""SELECT v.title as Video_title, v.duration as Video_duration, v.upload_time as Upload_time
            FROM video v, account a, upload u
            WHERE v.videoid = u.videoid AND a.accountid = u.accountid
            AND a.accountid = {acc_id}"""
```

## 5. Find all accounts that have more than N subscribers
The user is asked for the subscriber count they are looking for. 
A list of all account who have more than N subscribers is then displayed.
```python
    Q = f"""SELECT a.username Username, a.subscriber_count as sub_count
            FROM account a
            WHERE a.subscriber_count >= {sub_count}"""
```

## 6. Average advertisement revenue generated by an account per video
The user is asked for the accountID of the account whose revenue they would like to see.
A revenue report of the videos uploaded by that account is then displayed.
```python
    Q = f"""SELECT a.username us, v.revenue ar, v.title t
           FROM video v, upload u, account a
           WHERE u.videoID = v.videoID  AND a.accountID = u.accountID
           AND u.accountID = {acc_ID}"""
```

## 7. Search for an account
The user is asked for a keyword from the account they are looking for. A list of all account usernames containing that keyword is then displayed.
```python
    Q = f"""SELECT a.username as Username
            FROM account a
            WHERE a.username LIKE '%{keyword}%'"""
```

## 8. Get like-dislike ratio of a certain account's videos
The user is asked for the accountID of the account whose total like:dislike ratio they want to see.

```python
    Q = f"""SELECT SUM(video.likes) as likes, SUM(video.dislikes) as dislikes, account.username as us
            FROM video, upload, account
            WHERE video.videoID = upload.videoID AND account.accountID = upload.accountID
            GROUP BY upload.accountID
            HAVING upload.accountID = {acc_ID}"""
```


## 9. Get all videos with more than a million views
A simple ordered query result is displayed.
```python
    Q = f"""SELECT title, view_count, RANK() OVER (ORDER BY view_count DESC) as ord
            FROM video
            WHERE view_count > 1000000
            ORDER BY view_count DESC"""
```

## 10. Total watchtime of an account
The user is asked for the accountID of the account whose total watchtime they want to see.
```python
    Q = f"""SELECT SUM(watch_time), username
            FROM account
            WHERE accountID = {acc_ID}"""
```

## 11. Video with the most and least views on an account
The user is asked for the accountID of the account whose stats they want to see.
```python
    Q = f"""SELECT videoID
            FROM upload
            WHERE accountID = {acc_ID}"""
	Q1 = f"""WITH t_MAX AS
			(
			SELECT MAX(v.view_count) a_min
			FROM video v, upload u
			WHERE u.accountID = {acc_ID}
			)
			SELECT v.title t, v.view_count c
			FROM video v left join upload u on v.videoID = u.videoID
			WHERE u.accountID = {acc_ID}
			AND v.view_count IN (SELECT * FROM t_max)"""

	Q2 = f"""WITH t_MIN AS
			(
			SELECT MIN(v.view_count) a_min
			FROM video v, upload u
			WHERE u.accountID = {acc_ID}
			)
			SELECT v.title t, v.view_count c
			FROM video v left join upload u on v.videoID = u.videoID
			WHERE u.accountID = {acc_ID}
			AND v.view_count IN (SELECT * FROM t_min)"""
```

## 12. Search for a video
The user is asked for a keyword from the video they are looking for. A list of all video titles matching that keyword is then displayed.
```python
    Q = f"""SELECT title, a.username
            FROM
            video v left join upload u
                on v.videoID = u.videoID
            left join account a
                on u.accountID = a.accountID
            WHERE v.title LIKE '%{keyword}%'"""
```

## 13. Generate revenue report for an account
The user is asked for the accountID of the account whose revenue report they want to see.
```python
# videowise:
    Q = f"""SELECT v.title t, v.revenue r
            FROM account a JOIN upload u
                ON a.accountID = u.accountID
            JOIN video v
                on u.videoID = v.videoID
            WHERE a.accountID = {acc_ID}"""
            
# total revenue:
    Q = f"""SELECT username n, sum(v.revenue) s
            from account a JOIN upload u
                on a.accountID = u.accountID
            JOIN video v
                on u.videoID = v.videoID
            WHERE a.accountID = {acc_ID}
            GROUP BY username"""
```

## 14. Find engagement statistics for videos
The user is asked for the videoID of the video whose engagement statistics they want to see.
```python
# title, avg engagement rate:
    Q = f"""SELECT v.videoID vd, v.title t, avg(ew.engagement_rate_ad) r
            FROM engage_with ew JOIN video v
                ON v.videoID = ew.videoID
            WHERE v.videoID = {vid_ID}
            GROUP BY v.videoID"""

# video-wise ads.
    Q = f"""SELECT a.adID ad, a.advertiser av, ew.engagement_rate_ad ar
            FROM engage_with ew JOIN video v
                ON v.videoID = ew.videoID
            JOIN advertisement a
                ON a.adID = ew.adID
            WHERE v.videoID = {vid_ID}"""
```