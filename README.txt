*         ***     *     ***   ****   ****    ***   ****    ***   *   *  ****   *
*        *   *   * *   *   *  *   *  *   *  *   *   *  *  *   *  *   *  *   *  *
*         *     *   *   *     *   *  *   *  *   *   *  *   *     *   *  *   *  *
*          *    *****    *    ****   ****   *   *   ***     *    *   *  ****   *
*           *   *   *     *   *      * *    *   *   *  *     *   *   *  *      *
*        *   *  *   *  *   *  *      *  *   *   *   *  *  *   *  *   *  *      *
*         ***   *   *   ***   *      *   *   ***   ****    ***    ***   *      *;
/*			                   Generated with proc explode.                        */

/* Author: Daniel Mattei. https://www.linkedin.com/in/daniel-mattei-392929137/ */
/* Maintainer: Daniel Mattei <DMattei@live.com>			                  	       */
/* Version 1.0							                                           	       */

/* This program is an implementation of RProbSup's functionality in SAS.  	             */
/* It is designed for those interested in computing variants of the A statistic, a       */
/* non-parametric form of the common language (CL) effect-size statistic.                */
/* Although this program uses SAS/IML, it is designed for those most comfortable         */
/* With Base SAS - No interaction with SAS/IML is required, simply the macro program     */
/* provided and a few macro variables. However, the SAS/IML functions called by the      */
/* macro were also designed themselves to be called directly. 			                   	 */
/* A dataset from the DATA step is the expected input for the dsn= parameter of the      */
/* macro program.            								                                             */
/* Results are exported as a .sas7bdat file with output displayed for immediate viewing. */
/* Required Licenses: SAS/STAT | SAS/IML                                                 */

/* Special thanks to Rick Wicklin for his wonderful blog entries on the SAS/IML language */
/* and all of its neat features. Check out his blog for IML tips:                        */
/* https://blogs.sas.com/content/iml/                                                    */
/* Enthusiastic thanks to Dr. John Ruscio and his original RProbSup package,             */
/* which this program implements for use in SAS.                                         */
/* This package as well as other programs are accessible at John's website below:        */
/* https://ruscio.pages.tcnj.edu/quantitative-methods-program-code/                      */
/* For a discussion of the A statistic and its variants, please refer to                 */
/* Ruscio & Gera (2013)	 						                                                 		 */


/* What follows is a specification of the values and formats the parameters of the macro */
/* program will accept. 							                                                 	 */

/* Any membership variables (and any columns in general)  		                      		 */
/* should consist of numeric type data. 		                                    				 */

/* %A Macro Arguments:	                                                               	 */

/* dsn: The Base SAS dataset to be analyzed.					                                 	 */
/* For a between-subjects design, data should be in narrow/long format with the          */
/* membership variable as the first column.				                                   	 	 */
/* For a within-subjects design, data should be in wide format, with each observation    */
/* representing a person and each column representing a variable with their score.       */

/* design: Specification of the type of analysis designed.				                       */
/* 1 = between-subjects analysis (DEFAULT). 					                                 	 */
/* 2 = within-subjects analysis. 						                                           	 */

/* statistic: The version of the A statistic to be calculated.			                  	 */
/* 1 = The A statistic for 2 groups (DEFAULT).	      				                           */
/* 2 = The A statistic for k groups through the average absolute deviation.	             */
/* 3 = The A statistic for k groups through the average absolute pairwise deviation.	   */
/* 4 = The A statistic for k groups, comparing a reference group with the union of all 	 */
/*     other groups. 							                                                  		 */
/* 5 = The A statistic for the ordinal comparison of k groups.	                   			 */

/* weights: The weights to be assigned to each group.			                               */
/* 0 = The data are unweighted (DEFAULT).						                                     */
/* 1 = The data are are weighted. 		                                        					 */
/* NOTE: If 1 is entered, the weights should be in the final column.		              	 */

/* increase: Establishes whether scores are expected to correlate with the dummy coding  */
/*    	     of groups.						                                                			 */
/* NOTE: This argument is only used if statistic = 5.				                           	 */
/* 0 = False (DEFAULT). 					                                                 			 */
/* 1 = True.									                                                         	 */

/* ref: Designates the reference group of comparison.					                           */
/* NOTE: This argument is only used if statistic = 4.					                           */
/* 1 = The first group is the reference group (DEFAULT).			                         	 */

/* r: A row vector of proportions that specifies the proportions of the sample base      */
/*    rates of each group. If specified, the order of the proportions should correspond  */
/*    to the order of the group codes if between-subjects, or variable columns if        */
/*    within-subjects. In the macro program, r is entered delimited by a single space.	 */
/* NOTE: This argument is only used if statistic = 2.				                             */
/* 0 = Proportions are equal (DEFAULT).							                                     */

/* n_bootstrap: The number of samples generated to construct the bootstrapped sampling   */
/*		distribution.							                                                      	 */
/* 1999 = 1,999 samples are used (DEFAULT).					                                  	 */

/* conf_level: The confidence level used to construct the bootstrapped confidence 	     */
/*	       interval.					                                                    			 */
/* .95 = Confidence level of 95% (DEFAULT).				                                   		 */

/* ci_method: The method used to construct the bootstrapped confidence interval.	       */
/* 1 = Bias-corrected and Accelerated (BCa) bootstrap interval (DEFAULT).		             */
/* 2 = Percentile Interval bootstrap interval.					                               	 */

/* seed: Input for pseudo-random number generation.		                             			 */
/* 1 = The seed input (DEFAULT).						                                           	 */

/* missing: Determines whether missing values are removed or replaced by a constant.	   */
/* 0 = Missing values are removed (DEFAULT).		                                 				 */
/* 1 = Missing values are replaced by the number specified in mconstant=.                */

/* mconstant: Determines the constant to replace missing values with.		               	 */
/* NOTE: This argument is only used if missing = 1.					                             */
/* 0 = Missing values are replaced with 0 (DEFAULT).				                          	 */

/* ID: Designates whether the first column includes an ID variable that should be 	     */
/*     disregarded. 								                                                   	 */
/* 0 = The first column (ID variable) should be disregarded (DEFAULT).			             */
/* 1 = The first column does not include an ID variable that should be disregarded.      */
/* NOTE: If a between-subjects analysis is desired and ID = 0, the second column should  */
/*       contain the membership codes.							                                     */
/*       If the ID variable is the stratification variable for a between-subjects        */
/*       analysis, then argument should be set to 1. 				                          	 */

/* LASTLY: Note that the SAS/IML functions called by the macro program are designed to   */
/* 	       be used independently if desired. The arguments detailed above differ only in */
/*         the following way for these functions: A1() | CalcA1() | A2() | CalcA2()      */
/*         The arguments for the input of data are split into y1 and y2. Each should be  */
/*         row vectors.									 */
/*         In addition, some functions will have parameters that specify default weight  */
/*	       values. These are meant to prevent errors in certain instances if weights = 1 */
/*         is mistakenly selected without a weight column in the input data and may be 	 */
/*         ignored.					      			                                                 */
