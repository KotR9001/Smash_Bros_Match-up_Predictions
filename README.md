# Smash_Ultimate_Match-up_Predictions

This project was undertaken to make predictions about the overall match-up scores of the DLC characters in Smash 4.
The following will walk you through an easy to follow step by step analysis and thought process as we used a variety of programs to help 
read, analyze and display our findings and answer the questions below: 

1. What are the character parameters that best predict a character’s
performance as measured by the overall match-up score?    
-The overall character match-up score is the total of all individual     
match-up scores that a character has against all other characters (ex. Lucina vs. Fox, Link, Snake, etc.).

2. Which of the DLC characters is the best and why?    
-If certain traits are significantly more important than others and top characters consistently fit a certain archetype (based on stats), 
what can developers learn based on our model to better balance the game and ensure that no one type of character is too dominant?    
-As an example, if frame data is by far the best predictor of match-up scores, then would it be sensible to make fast characters deficient 
in most or all other parameters

Data Sources: 

-We had two main data sources that were used:

	1. API Character Data for DLC characters for Super Smash Bros.  
	- https://kuroganehammer.com/Ultimate
	2. Google Docs with match up data for each character
	- https://docs.google.com/spreadsheets/d/1QV2_WC--SEPUVM5U2qvgh_uJHKPhZzXAcDhDE6irtP4/edit#gid=1934011249
	

Cleaning the Data

-Initial process, make API calls and read json files.
 
-Followed by creating data frames from json so that cleaning process can be done. 

-API calls for jsons included data for attributes, moves, unique characteristics etc. Only data for moves & character attributes jsons were 
used, because movement json contained duplicate fields with character attributes, characters json only contained character names and IDs 
(also duplicates), and unique contained some miscellaneous data.

-After calls had been made, the following step was to create Pivot dataframes without aggregation.  This pivot is helpful to see our data 
in a different way - often turning a format with many rows that would require scrolling into a newformat with fewer rows but more 
columns. In this case it allowed each character to be in a row with the different characteristics in columns based on the json read.

-In addition to the API data, we also used a google doc containing matchup data for each character. The matchup chart is an amalgamation 
of every notable player’s matchup chart since 2016, only using data from 2017 and onwards. Averaging this all together, it creates a 55x55 
grid of matchup data, tier list based on matchup coverage, and various other stats.

-Our final step before continuing to our machine learning process was to create joins that would create the 'joined_final' dataframe that 
would be used in the following steps.
  
	-Outer and inner joins were used with each join having a specific lsuffix and rsuffix to help distinguish which column referenced 
	whatjson data.

	-The 'joined_final' table was exported as an Excel for use in Tableau.


Machine Learning

-First, the 'joined_final' dataframe was split into X (feature columns) and y (Overall Match-up Scores) variables. Each of these was split
to yield the following:

	X_train: X values for the initial release characters

	X_test: X values for the DLC characters

	y_train: y values for the initial release characters

	y_test: y values for the DLC characters

-Next, a method was used to fill in the missing X values, because most sklearn models don't work with missing data. Multiple methods were
tried, and ultimately, an IterativeImputer was chosen, since it allows a strategy of choice to be used to both use adjacent feature columns
to predict missing X values and do so with multiple passthroughs to iteratively improve the predictions.

-Since IterativeImputer works best with Gaussian data, StandardScaler was used to apply a transformation to center data at 0 with a standard
deviation of 1.

-After that, hyper-parameter tuning was done with IterativeImputer, because several initial attempts to generate predictions with our model
gave negative r^2 values.

	-Every combination of parameter values was tested.

	-However, since IterativeImputer doesn't work with GridSearchCV, it was instead embedded within a series of nested for loops in
	order to test all of these combinations.

	-The parameter values were as follows:
	max_iter=[5, 10, 20]
 
	n_nearest_features=[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28,
                   29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54,
                   55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
                   81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105,
                   106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126,
                   127, 128, 129, 130, 131, 132]
 
	initial_strategy=['mean', 'median']

	sample_posterior=[True, False]

	add_indicator=[True, False]

	-After obtaining the r^2 values for each combination of parameter values, it was found that the highest possible r^2 value was
	0.7041.

-Once hyper-parameter tuning was completed, the optimal values were assigned to their respective parameters in the IterativeImputer, and
it was run.

-Then, the model was trained with the X_train and y_train variables to calculate the correlation coefficients as in the hyper-parameter
tuning.

-Next, the r^2 value was obtained for the initial release characters and the DLC characters.

-Finally, the model was used to predict overall match-up scores for the DLC characters, with the predictions tabulated with the y_test data
to compare predicted vs actual values.

-This table was exported to a CSV for use in Tableau.


Display Results

-After conducting our machine learning, we opted to use Tableau as our visual format for dispalying our results. 
Link: https://public.tableau.com/profile/nonelaci.her#!/vizhome/SuperSmash4/ProjectPoint?publish=yes
