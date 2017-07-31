---
layout: post
title: First post - The Eight Queens Problem
---

For my first post on this blog I thought it'd be good to go through how I solved The Eight Queens problem, not because I've come up with a particularly elegant solution but just for purely selfish reasons: this is my first blog post and I want to start getting used to blogging!

The Eight Queens problem is a classic puzzle where the challenge is to find a solution where eight queens are placed on a chess board so that no queen can take another in a single move. My challenge was to solve this using good ole T-SQL. Itzik Ben-Gan posted this problem and a really nice solution a few years ago but I only came across it recently: <http://sqlmag.com/t-sql/use-t-sql-solve-classic-eight-queens-puzzle>.

Normally when I see posts like this that first state to try it yourself first before reading the solution, I normally cheat and just read the solution but this time round I thought I'd give it a whack and see what happens. Pleased to subsequently find out that the logic I used was fairly sound and I ended up with the same number of solutions!

I started off by recognising that I'd need a way of referring to each possible position on the 8 x 8 chessboard. Classic notation would be a1, b4 etc. but I thought I might run into conversion headaches later down the line so I just settled on x and y co-ordinates e.g. (3, 7) is the equivalent of c7.

I created the possible positions table by using a CTE and a cross join:

```SQL
CREATE TABLE [PossiblePositions]
	( [HorizPosNum] INT
	, [VertPosNum] INT
	);

WITH [HorizVert] AS (
	SELECT [xy]
	FROM (VALUES (1), (2), (3), (4), (5), (6), (7), (8)) AS CoOrds(xy) 
) 
INSERT INTO [PossiblePositions]
	SELECT 
		HV1.[xy]
		, HV2.[xy]
	FROM [HorizVert] HV1
	CROSS JOIN [HorizVert] HV2;
```
