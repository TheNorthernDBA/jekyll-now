---
layout: post
title: First post - The Eight Queens Problem
---

For my first post I thought it'd be good to go through how I solved The Eight Queens problem, not because I've come up with a particularly elegant solution but just for purely selfish reasons: this is my first blog post and I want to start getting used to blogging!

The Eight Queens problem is a classic puzzle where the challenge is to find a solution where eight queens are placed on a chess board so that no queen can take another in a single move. My challenge was to solve this using good ole T-SQL. Itzik Ben-Gan posted this problem and a really nice solution a few years ago but I only came across it recently: <http://sqlmag.com/t-sql/use-t-sql-solve-classic-eight-queens-puzzle>.

Normally when I see posts like this that first state to try it yourself first before reading the solution, I normally cheat and just read the solution but this time round I thought I'd give it a whack and see what happens. Pleased to subsequently find out that the logic I used was fairly sound and I ended up with the same number of solutions... I'd urge you to do the same if you have the time and if you get a kick out of solving these sort of things!

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

Also created a table to store the final list of solutions, this would be a table with 16 `int` columns; an x and y column for each queen.

```SQL
CREATE TABLE [Solutions]
	( [Q1X] INT
	, [Q1Y] INT
	, [Q2X] INT
	, [Q2Y] INT
	, [Q3X] INT
	, [Q3Y] INT
	, [Q4X] INT
	, [Q4Y] INT
	, [Q5X] INT
	, [Q5Y] INT
	, [Q6X] INT
	, [Q6Y] INT
	, [Q7X] INT
	, [Q7Y] INT
	, [Q8X] INT
	, [Q8Y] INT
	);
```

The next bit took a bit of head scratching but I needed to figure out what would constitute a false solution, i.e. a solution where one queen could feasibly take another. This means ruling out any queens that share a horizontal, vertical or diagonal line. Horizontal and vertical lines are easy from a logical perspective, they're just the ones that share an x or y co-ordinate. 

The diagonal line took a bit more thinking about but if you think about it in algebraic terms and consider a queen at position (x, y) then all of the diagonal positions are displaced by the same amount in the x and y plane denoted by (x ± n, y ± n). So, for example, take the position (3, 6) and n equal to 2, then (5, 8) is on a diagonal line, as is (5, 4). Or, with the queen in the same position and n equal to 4, then (7, 2) is also on the diagonal.

We can generalise this by saying that if the absolute value of the difference between the x co-ordinates and the absolute value of the difference between the y co-ordinates of 2 queens is the same, then those 2 queens are diagonally aligned. Or mathematically, for Q1(a, b) and Q2(c, d), if ABS(a - c) = ABS(b - d) then Q1 and Q2 are in the same diagonal.

Just a quick recap then, as far as I was concerned for any Q(x, y), incorrect solutions would contain co-ordinates for another queen where it's co-ordinates match one of these three patterns:
1. Q(x, y +- n) (same x co-ordinate)
2. Q(x +- n, y) (same y co-ordinate)
3. Q(x +- n, y +- n) (on the diagonal)

I figured from this I could just populate the Solutions table with a bunch of possible solutions then remove any that are false. Made a quick backtrack when I realised that I can easily rule out 1 and 2 in the first insert and avoid having a huge table for starters. 1 and 2 also mean (in a roundabout way), that for 8 queens on an 8 x 8 grid, for every solution, there will always be exactly one queen with each x co-ordinate (1 through 8) and exactly one queen with each y co-ordinate (1 through 8), they'll just be mixed around a bit!

Here's the code I used to do the initial populate of the solutions table:

```SQL
INSERT INTO [Solutions]
	SELECT Q1.[HorizPosNum]
		, Q1.[VertPosNum]
		, Q2.[HorizPosNum]
		, Q2.[VertPosNum]
		, Q3.[HorizPosNum]
		, Q3.[VertPosNum]
		, Q4.[HorizPosNum]
		, Q4.[VertPosNum]
		, Q5.[HorizPosNum]
		, Q5.[VertPosNum]
		, Q6.[HorizPosNum]
		, Q6.[VertPosNum]
		, Q7.[HorizPosNum]
		, Q7.[VertPosNum]
		, Q8.[HorizPosNum]
		, Q8.[VertPosNum]
	FROM [PossiblePositions] Q1
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 2) Q2
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 3) Q3
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 4) Q4
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 5) Q5
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 6) Q6
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 7) Q7
		CROSS JOIN (SELECT [HorizPosNum], [VertPosNum] FROM [PossiblePositions] WHERE [HorizPosNum] = 8) Q8
	WHERE Q1.[HorizPosNum] = 1
		AND Q2.[VertPosNum] NOT IN (Q1.[VertPosNum])
		AND Q3.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum])
		AND Q4.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum], Q3.[VertPosNum])
		AND Q5.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum], Q3.[VertPosNum], Q4.[VertPosNum])
		AND Q6.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum], Q3.[VertPosNum], Q4.[VertPosNum], Q5.[VertPosNum])
		AND Q7.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum], Q3.[VertPosNum], Q4.[VertPosNum], Q5.[VertPosNum], Q6.[VertPosNum])
		AND Q8.[VertPosNum] NOT IN (Q1.[VertPosNum], Q2.[VertPosNum], Q3.[VertPosNum], Q4.[VertPosNum], Q5.[VertPosNum], Q6.[VertPosNum], Q7.[VertPosNum])
```

This creates a table with 40,320 solutions including all the ones where queens share diagonals so the next step is to get rid of those:

```SQL
DELETE FROM [Solutions]
WHERE ABS(Q2Y - Q1Y) - ABS(Q2X - Q1X) = 0
	 OR ABS(Q3Y - Q1Y) - ABS(Q3X - Q1X) = 0 OR ABS(Q3Y - Q2Y) - ABS(Q3X - Q2X) = 0
	 OR ABS(Q4Y - Q1Y) - ABS(Q4X - Q1X) = 0 OR ABS(Q4Y - Q2Y) - ABS(Q4X - Q2X) = 0 OR ABS(Q4Y - Q3Y) - ABS(Q4X - Q3X) = 0
	 OR ABS(Q5Y - Q1Y) - ABS(Q5X - Q1X) = 0 OR ABS(Q5Y - Q2Y) - ABS(Q5X - Q2X) = 0 OR ABS(Q5Y - Q3Y) - ABS(Q5X - Q3X) = 0 OR ABS(Q5Y - Q4Y) - ABS(Q5X - Q4X) = 0
	 OR ABS(Q6Y - Q1Y) - ABS(Q6X - Q1X) = 0 OR ABS(Q6Y - Q2Y) - ABS(Q6X - Q2X) = 0 OR ABS(Q6Y - Q3Y) - ABS(Q6X - Q3X) = 0 OR ABS(Q6Y - Q4Y) - ABS(Q6X - Q4X) = 0 OR ABS(Q6Y - Q5Y) - ABS(Q6X - Q5X) = 0
	 OR ABS(Q7Y - Q1Y) - ABS(Q7X - Q1X) = 0 OR ABS(Q7Y - Q2Y) - ABS(Q7X - Q2X) = 0 OR ABS(Q7Y - Q3Y) - ABS(Q7X - Q3X) = 0 OR ABS(Q7Y - Q4Y) - ABS(Q7X - Q4X) = 0 OR ABS(Q7Y - Q5Y) - ABS(Q7X - Q5X) = 0 OR ABS(Q7Y - Q6Y) - ABS(Q7X - Q6X) = 0
	 OR ABS(Q8Y - Q1Y) - ABS(Q8X - Q1X) = 0 OR ABS(Q8Y - Q2Y) - ABS(Q8X - Q2X) = 0 OR ABS(Q8Y - Q3Y) - ABS(Q8X - Q3X) = 0 OR ABS(Q8Y - Q4Y) - ABS(Q8X - Q4X) = 0 OR ABS(Q8Y - Q5Y) - ABS(Q8X - Q5X) = 0 OR ABS(Q8Y - Q6Y) - ABS(Q8X - Q6X) = 0 OR ABS(Q8Y - Q7Y) - ABS(Q8X - Q7X) = 0

```

Pretty ugly right? But this is just a long-hand way of getting rid of all the rows that meet condition 3. I'm sure given a bit more time I could be a bit cleverer about this and do it programattically but anyhoo... a quick random check of some of the results and they seem correct...

![alt text](http://thenortherndba.github.io/images/8Qs.jpg "Sanity check")

Double checking Itzik's post and... Ta-da! 92 results, the same as the solution Itzik posted. But mine was just done in a far more brute force kind of fashion. Funny to see how Itzik has posted a far more elegant solution but I'm glad I gave it a go!
