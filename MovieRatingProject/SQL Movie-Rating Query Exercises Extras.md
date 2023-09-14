### Your queries are executed using SQLite, so you must conform to the SQL constructs supported by SQLite.

1. Find the names of all reviewers who rated Gone with the Wind.
```sql
SELECT distinct(R.name)
FROM Reviewer R, Rating RA
WHERE R.rid = RA.rid and RA.mid in (SELECT M.mid FROM Movie M WHERE M.title = "Gone with the Wind");
```

2. For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars.
```sql
SELECT R.name, M.title, RA.stars
FROM Movie M, Reviewer R, Rating RA
WHERE R.name = M.director and R.rID = RA.rID and M.mID = RA.mID;
```

3. Return all reviewer names and movie names together in a single list, alphabetized. (Sorting by the first name of the reviewer and first word in the title is fine; no need for special processing on last names or removing "The".)
```sql
SELECT R.name
FROM Reviewer R
UNION
SELECT M.title
FROM Movie M
ORDER BY R.name, M.title;
```

4. Find the titles of all movies not reviewed by Chris Jackson.
```sql
SELECT M.title
FROM Movie M
WHERE M.mID not in
(SELECT RA.mID
FROM Rating RA
WHERE RA.rID in
(SELECT R.rID
FROM Reviewer R
WHERE R.name = "Chris Jackson"));
```

5. For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once. For each pair, return the names in the pair in alphabetical order.
```sql
SELECT DISTINCT 
    MIN(R.name, R2.name) name1,
	MAX(R.name, R2.name) name2
FROM Reviewer R, Reviewer R2, Rating RA1, Rating RA2
WHERE R.rID <> R2.rID and RA1.rID = R.rID and RA2.rID = R2.rID and 
RA1.mID = RA2.mID
GROUP BY RA1.mID
ORDER BY name1, name2;
```

6. For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars.
```sql
SELECT R.name, M.title, RA.stars
FROM Rating RA JOIN Movie M on RA.mID=M.mID
JOIN Reviewer R on RA.rID = R.rID
WHERE RA.stars = (SELECT MIN(stars) FROM Rating);
```

7. List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order.
```sql
SELECT M.title, AVG(RA.stars)
FROM Movie M, Rating RA
WHERE M.mID=RA.mID
GROUP BY M.title
ORDER BY AVG(RA.stars) DESC, M.title;
```

8. Find the names of all reviewers who have contributed three or more ratings. (As an extra challenge, try writing the query without HAVING or without COUNT.)
```sql
SELECT Rev.name
FROM Reviewer Rev JOIN
(SELECT Rev.name, COUNT(RA.rID) AS Ratings
FROM Reviewer Rev JOIN Rating RA ON Rev.rID = RA.rID
GROUP BY Rev.name) Rev2 ON Rev.name = Rev2.name
WHERE Ratings>=3;
```

9. Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title. (As an extra challenge, try writing the query both with and without COUNT.)
```sql
SELECT M.title, M.director
FROM Movie M
WHERE M.director in (SELECT M2.director FROM Movie M2 WHERE M2.mID <> M.mID)
ORDER BY M.director, M.title;
```

10. Find the movie(s) with the highest average rating. Return the movie title(s) and average rating. (Hint: This query is more difficult to write in SQLite than other systems; you might think of it as finding the highest average rating and then choosing the movie(s) with that average rating.)
```sql
SELECT M.title, MAX(average)
FROM Movie M JOIN (SELECT RA.mID, AVG(RA.stars) AS average
FROM Rating RA
GROUP BY RA.mID) averageRating ON M.mID = averageRating.mID;
```

11. Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating. (Hint: This query may be more difficult to write in SQLite than other systems; you might think of it as finding the lowest average rating and then choosing the movie(s) with that average rating.)
```sql
SELECT M.title, AVG(RA.stars) as AV
FROM Movie M JOIN Rating RA ON M.mID = RA.mID
GROUP BY M.title
HAVING AVG(RA.stars) = (SELECT MIN(average)
FROM (SELECT AVG(RA.stars) AS average
FROM Rating RA JOIN Movie M ON RA.mID = M.mID
GROUP BY M.title) subq);
```

12. For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL.
```sql
SELECT M.director, M.title, RA.stars
FROM Movie M JOIN Rating RA ON M.mID= RA.mID
WHERE M.director is not NULL
GROUP BY M.director
HAVING RA.stars = MAX(RA.stars);
```


















