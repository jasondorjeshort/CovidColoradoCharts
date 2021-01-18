# CovidColoradoCharts
Charts from Colorado's COVID data

![Cases, Hospitalizations, and Deaths by infection day](https://raw.githubusercontent.com/jasondorjeshort/CovidColoradoCharts/main/cases-hospitalizations-deaths-tests-infection-log.png)

![CFR by infection day](https://raw.githubusercontent.com/jasondorjeshort/CovidColoradoCharts/main/CFR-infection.png)

![CHR by infection day](https://raw.githubusercontent.com/jasondorjeshort/CovidColoradoCharts/main/CHR-infection.png)

![HFRby infection day](https://raw.githubusercontent.com/jasondorjeshort/CovidColoradoCharts/main/HFR-infection.png)

![Positivity by infection day](https://raw.githubusercontent.com/jasondorjeshort/CovidColoradoCharts/main/Positivity-infection.png)

# Simple Explanation

To track true epdidemic data, we'd like to look at data by day of infection.  Most sources will only give you numbers by day of public release, but this can be delayed by days, weeks, or even months.  This data appears to be "up to date", but each value is just a snapshot of a subset of values from a range of dates in the past: it's being obfuscated!  Colorado is unusual in that the state tracks most numbers by day of symptom onset.  This may be imperfect and incomplete in several ways, but by subtracting 5 days we can get a better estimate of epidemic data by day of infection than is available in any other way.

The issue with these numbers is that they are always incomplete.  Cases with an infection date of yesterday, hospitalizations infected a week ago, or deaths infected two weeks ago will almost certainly not be reported yet.  One common approach, used by the CDPHE dashboard, is to pick an arbitrary cutoff in the past in which to mark numbers as "inaccurate".  But this doesn't work very well: numbers lag by a greater or lesser amount depending on test returns, and even more importantly there is really no upper bound on which they can change.  Colorado routinely adds new numbers from months ago, with a few outliers from as much as a year in age.  In generating these graphs, I have taken a simple but effective approach to combatting this problem: tracking a rolling average of how much the numbers increase over time and "projecting" them up to the present.

The graphs shown here represent a few things.  Tests, cases, hospitalizations, and deaths are provided by infection date. The "rates" graphs are merely divisions of two of these values, expressed as a percentage.  Most charts are expressed as an average over time.

# Read On

Although analytics are used here, it's important to note that this is not a model predicting the future.  I use logarithmic graphs because contagious diseases spread exponentially, so on a log graph they should almost always end up as a sequence of line segments.  It has been demonstrated that the best possible model predicting future spread is actually just choosing two points on the log graph and drawing a line; use this model to predict the future if you dare.

A central use of this curve is to estimate what actions we have done that affect the epidemic curve.  When we view the true epidemic curve by day of infection, the results of an action should be visible (eventually) on the very day it happened.  Several of the "events" shown on the graph were chosen as things that obviously affected spread: the day Governor Polis re-closed bars, and the day he mandated masks, were chosen because they obviously should change the curve - and they did.  Several other events on the graph (the stay-at-home order, the beginning of the Black Lives Matter protests, the state's first snowfall, Thanksgiving, Christmas, and the day around half of the state moved to "red" status) are included because they seem significant, but the effects may be uncertain.  Thirdly I have pinned two days as the start and end of the "November wave", which correspond identically to the opening and closing of Denver Public Schools - but this is chosen retrospectively, and could just be conincidence.  Finally, V+5 is the 5th day after vaccinations started, which (when measured by day of infection) should represent the start of a roughly linear decline in hospitalization and mortality rates (but this data is still incomplete).

# The Math

Read on at your mathematical peril.  CDPHE gives us data archives for each day, and each day includes cases, hospitalizations, and deaths by date of onset.  Subtracting 5 gives us a good estimation of date of infection.  We end up with a three-dimensional set of data. Dimension 1 is the "day of data", and is the released numbers for each day.  Dimension 2 is the "day of infection" (or more generally, the "day of type"): for each day of data, we have an entirely new number for each day of infection.  Dimension 3 is the numbers (e.g., cases) themselves.  The graphs here show only the most recent snapshot of the most recent day of data (though projections are calculated from each previous day); representing all three dimensions is done in the software by creating an animation.

If we track a day of infection through its consecutive days of data, we (usually) see it increasing.  For example, through days 0 through 5 after infection it may remain at 0.  Then it jumps to 5, then 10 (x2), then 30 (x3), then 100 (x3.3), then 120 (x1.2), then 140 (x1.17), and so on.  This exemplifies the delay between infection and data: for each number of delay and each day of data, we have a multiplicative ratio (also a 3-dimensional data set).  What I do is track the ratio for each "delay" value, and take a moving average of those daily numbers.  To be precise, I use a Kalman geometric average with (basically arbitrary) weighting 14+delay for each day.  At the end, final projections can be found by muliplying together all of these ratios for each incomplete day.

Finding test numbers is a completely different calculation.  On each day of data, new cases are added for each day of infection, and new tests are added as just a single total for the day.  A positivity value for that day of data can easily be calculated, and dividing the number of new cases for each day of infection by that positivity can give us an estimate of the tests for that day of infection.  This test value by itself isn't particularly noteworthy, though it is important that the total number of tests is not changed by this algorithm - we're simply moving from them around from the day of data to the day of infection.  This does allow us to get a positivity estimate by day of infection, however.

If you believe you should understand this explanation, but do not, I urge you to contact me with questions.  Mathematically, the workings here are intended to be intuitive and visualizable.  This is a model of how the numbers should work; there is no fitting to how they do work.  The only failure, I believe,  is that I should be tracking 95% confidence intervals rather than rolling averages.

# The Programming

The code here is quite ugly.  I literally just picked java because that's what I wrote my last program in, googled to find a chart program, and then went to work.  The code is included as an Eclipse project - as the data is included as a sequence of CSV's - in sister github projects.  There is a lot more that is visualized with the charts created by the program - animated GIFs show the numbers over time, numbers can be seen by date of infection, onset, reporting, or death (all of which are incomplete), county final numbers are available (but not by infection day, so these are no different than any other data source), the average age from infection/onset/reporting/death to CDPHE release can be found, and...so on.