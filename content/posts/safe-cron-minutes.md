+++
title = "Safe cron minutes"
date = 2024-03-07 12:00:00
+++

	06 07 08 09 11 12 13 17
	18 19 21 22 23 24 26 27
	33 34 36 37 38 39 41 42
	43 47 48 49 51 52 53 54

## Theory
Everyone runs their scheduled jobs at simple minute steps like 60, 30, 15, or 5.

	   0 * * * *
	*/30 * * * *
	*/15 * * * *
	*/ 5 * * * *

## Problem
Cron aligns minutes to the clock, leading to huge spikes of usage at those
intervals, as the hourly jobs, half-hourly jobs, etc, all run at once.

    |                       |                       |                       |
    |           |           |           |           |           |           |
    |     |     |     |     |     |     |     |     |     |     |     |     |
    | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | | |
	|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||

## Solution
Use random, unusual minutes to distribute the load. Exclude the most common
minutes, and nearby minutes based on prevalence.

	7 * * * *
	26 */2 * * *
	34,53 * * * *

## Rules

Begin with the set of all minutes in the hour.

1. Exclude 5 minute intervals.
2. Exclude 15 minute intervals and nearest 1 minutes.
3. Exclude 30 minute intervals and nearest 2 minutes.
4. Exclude 60 minute intervals and nearest 4 minutes.
5. Select minutes randomly from the remaining set.

<!-- -->

	1.	2.	3.	4.	5.

	00
		01
			02
				03
				04
	05
					06 <
					07 <
					08 <
					09 <
	10
					11 <
					12 <
					13 <
		14
	15
		16
					17 <
					18 <
					19 <
	20
					21 <
					22 <
					23 <
					24 <
	25
					26 <
					27 <
			28
		29
	30
		31
			32
					33 <
					34 <
	35
					36 <
					37 <
					38 <
					39 <
	40
					41 <
					42 <
					43 <
		44
	45
		46
					47 <
					48 <
					49 <
	50
					51 <
					52 <
					53 <
					54 <
	55
				56
				57
			58
		59
	00

## Conclusion
Is this really problem? Probably not; better solutions have likely taken care of
it.

Don't take `19 * * * *`, though. I'm using it.
