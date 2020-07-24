# Working with dates.  
###[Click here for weblink](http://biostat.mc.vanderbilt.edu/wiki/pub/Main/ColeBeck/datestimes.pdf)
***

###### Dates are often thought of as one of the most tedious task to deal with when working in R. Be it data importing, data wrangling or data visualization, warnings and errors seem to be a frquent pop ups on our R interface. No matter how irritating they sound, time-series analysis is still an important aspect of data analysis. So this post deals with understanding of some of the basic concepts related to the use of "Date-Time" in R.  

## 1. Using dates as integers depending on the type of work and employing a lookup table.

### 1.1. Match() function basic
Sometimes there is no need of dates directly in the code, rather an integer value (corresponding to each date/date-time) is sufficient to index the data such that one can look up on the date-time as and when required. In such a case,  one can use a lookup table.  
For that, firstly, a lookup table is created. For instance, `date.lookup` as follows


	date.lookup <- format(seq(as.Date("2000-01-01"), as.Date("2010-12-31"), by = "1 day"));
	class(date.lookup);  # "character" class

The **match()** function is then used to find the position of a particular date. 

	match("2004-01-19", date.lookup);
	match("2012-01-01", date.lookup);   # output is NA because value is not present in lookup table.

The lookup table is actually a character vector and not a date vector. So, basically what `match(x,table)` function does is, it looks for a character `x` in the lookup table `table`. Looking for a date which is not in the lookup table returns **NA**. An alternate function `%in%` is also there in place of `match()` but not explained in this discussion.  
An additional point is that if date.lookup had been created as a date vector, `match()` wouldn't have worked. You can try the same by using the following.

	date.lookup <- seq(as.Date("2000-01-01"), as.Date("2010-12-31"), by = "1 day");
	class(date.lookup);    # "Date" class
	match("2004-01-19", date.lookup);    # output is NA


Now, other than locating the date in the lookup table, one can also call the date value from the index value. However, going beyond the length of lookup table returns **NA**.
 
	date.lookup[1001];
	date.lookup[10000];   # output is NA as the index value is out of range
	
  
### 1.2. Match function for vectors and matrices
If we use **match()** function for vectors (as in case of above example), it returns a value as per the position of the `x` in the `table`.  
However, matrices are two dimensional arrays in which the position must be given by an x and y coordinate. But match() function, on the contrary, locates the position by counting method only just like vectors. So, the position is counted column wise i.e. first the elements of column 1 are counted and then we move to the next column and so on.

	vec <- c(1:20, 40:60);
	match(41, vec);
	mat <- matrix(c(1:20, 41:60), nrow = 5);
	match(45, mat);

### 1.3. Match function for dataframes.
If we try to use the **match()** function for dataframes, it returns **NA**.

	df <- data.frame(A= 1:10, B= 11:20, C= 21:30, D= 31:40);
	match(21, df);

The possible reason for this can be:  
*Matrices in R have elements which are all of same data type and since matrix is created from a single vector only by specifying the number of rows, it can be transformed back to a vector too. And this is what probably match() function might be doing. It might be converting back the matrix to a vector and give the location by counting. However, this is not possible in dataframes because neither the elements in the dataframe are necessarily of the same data type nor they are in some way made from a vector. So the developers might have understood this and thus kept the match() function out of the dataframe class.*


## Ex.1 Finding whether your child is telling the truth or lie?
Let us say, your child had a number of Maths tests conducted on his tuition over a period of time. He tells you that he scored more than 95% of marks on all the dates as listed in `son_dates`. You also have a list of all the dates on which tests were conducted as sent to you by the teacher `teacher_dates`. Your strategy is to to match the former to the latter in order to see whether the test was actually conducted on that particular date or not. But to avoid any mistake and do it quickly, you plan to write a code.

	son_dates <- c("2019-01-01", "2019-01-29", "2019-02-18", "2019-03-04", "2019-03-26");
	teacher_dates <- format(seq(as.Date("2019-01-01"), as.Date("2019-03-31"), by = "1 week"));
	match(son_dates, teacher_dates);     # output has two NA values suggesting son was lying 



## 2. Using dates as DATE class.
  
### 2.1. Creating a date. 
**as.Date()** function is used to create a date from a character vector and a date by default should preferably be created in no format other than yyyy-mm-dd. If stored in other formats, the date is stored after coercing to the default format.  
For instance, in the following code,  
**date1** is stored as yyyy-mm-dd= 0015-06-15 i.e. *year = 15, month = 06 and date = 20*  
**date2** is stored as yyyy-mm-dd= 0020-06-15 i.e. *year = 20, month = 06 and date = 15*  
**date3** is the correct way of passing the argument.  


	date1 <- as.Date("15-06-2020");
	date2 <- as.Date("20-06-15");
	date3 <- as.Date("2020-06-15");
  
### 2.2. Formatting the date. 
Though date should preferably be stored in the default format but still R is flexible enough to allow you to enter the data in a different format as long as that format is mentioned by passing an additional argument `format = ` .

	date4 <- as.Date("2020-06-15", format= "%Y-%m-%d");
	date5 <- as.Date("06-15-2020", format= "%m-%d-%Y");
	date6 <- as.Date("15/06/2020", format= "%d/%m/%Y");
	date7 <- as.Date("200615", format= "%y%m%d");
	date8 <- as.Date("20201506", format= "%Y%d%m");

In the above cases, we tell the **as.Date()** function the format in which we have entered the date by passing another argument **format = ""** and then the **as.Date()** function reads the date in that format which we have provided and finally convert it into the default format *yyyy-dd-mm* which is returned as the output.  
The important thing to note is that **%y refers to yy** format while the **%Y refers to YYYY** format. Similarly, there are many such other formats, which can be looked upon  in the *Help* menu or using `?strftime`. It is explained in detail in Point 4 below.


### 2.3. Importance of Origin argument in as.Date() function  
Date objects are stored in R as integer values (*not decimal values*) and to visualise such numbers in the form of dates, we need an origin date with respect to which the other numbers are converted to dates. For instance,

	date9 <- as.Date(10, origin= "2020-01-01");   # output is "2020-01-11"

So what R does is, it takes the origin as Jan 1, 2020 and gives it the index 0 and then calculates the date with index 10 i.e. adding 10 days to Jan 1, 2020 thus giving Jan 11, 2020 as the output. Indexing in R starts from zero. So it can also be said that Dates in R are basically indices of the position of that particular date with respect to the origin; origin which has index = 0.  
The origin date used by **R** is by default **1970-01-01** and most of other data analysis tools also save date as numbers and have a default origin date already stored in them as in **R**. This origin can also be modified in **R** as already shown in the example above.  

In order to show that the dates actually behave like numbers when used in certain  (not all) arithimetic or logical operations, the following code is written. Note the numeric nature of dates is only applicable to certain functions. The *addition, multiplication operations etc* give an error when used for dates, however, *subtraction* works. Similarly *greater than and lesser than* logical operations also work.

	Sys.Date() - as.Date("01-01-2020", format= "%d-%m-%Y");
	as.Date("2020-01-01") + 10;
	as.Date(25, origin = "2020-01-01") + 10;

The dates can be subtracted using an alternative function **difftime()** which has the advantage of taking the difference in the user specified format i.e. in the form of *min, day, etc.* as shown.

	difftime(Sys.Date(), as.Date("01-01-2020", format= "%d-%m-%Y"), units = "days");
	difftime(Sys.Date(), as.Date("01-01-2020", format= "%d-%m-%Y"), units = "mins");

The argument `units` in `difftime()` can have values like:  **sec** or **secs**, **min** or **mins**, **hour** or **hours**, **day** or **days** and **week** or **weeks**. 


*To see the internal integer representation of date, use the following code:*

	unclass(Sys.Date());

## 3. POSIXt Date-Time class
The way one works with Date class data is the same way one works with Date-Time data. There are mainly 3 libraries in R used for working with date-time data: **default base package, chron package, lubridate package**. While it may be easy to work with additional packages sometimes but it is always better to know the default options available. Simultaneously, in a situation where you may have to write code for something for which someone has already written a code and created a package, in that case, its better to use the package instead.  
In the **base package**, there are two functions to work with date-time data namely, **as.POSIXct()** and **as.POSIXlt()**  

1. **ct** in **POSIXct** stands for **calendar time** and it stores the date in the form of an integer (as already mentioned) with the integer value representing the number of seconds since the origin.  
2. **lt** in **POSIXlt** stands for **local time** and it keeps the date as a list of time attributes i.e. it creates a list which has various time aspects like *weekday, month, hour, year etc.* stored in it with respect to the date passed in the argument.  

The following two codes show the behavior of the two functions.

		unclass(as.POSIXct(Sys.time()));
		unclass(as.POSIXlt(Sys.time()));

However, there are some key points related to `as.POSIXlt` function as follows:

1. Default **origin year** in this function is 1900 which is indexed as year 0 i.e. the period from Jan,1900 to Dec,1900 is termed as year 0, Jan,1901 to Dec,1901 as year 1 and so on.
2. Similarly, **second** 0-1 is considered as 0 and **second** 1-2 is considered as 1 and so on.
3. Similarly, **minute** 0-1 is considered as 0 and **minute** 1-2 is considered as 1 and so on.
4. Similarly, **hour** 0-1 is considered as 0 and **hour** 1-2 is considered as 1 and so on.
5. Similarly, **mon** Jan is considered as 0 and only after it has completed and we have moved on to February is when the **mon** changes 1.
6. Similarly, **yday** 0-1 is considered as 0 and **yday** 1-2 is considered as 1 and so on. 
7. However, **wday** and **mday** consider the period which is going on i.e. **wday** for Monday is 1 and **mday** for 23 Jan is 23.
8. **$isdst** is a logical vector which tells whether daylight saving is on or off.
9. **$gmtoff** is the seconds by which the time zone is ahead or behind of **GMT**. For India, **$gmtoff** is 19800 i.e. (19800/3600) = 05:30 hours.  

These attributes of `POSIXlt` are of special importance in certain cases as in the `SEASONS()` function developed by me and discussed in some other post.


## 4. Different Date-Time formats discussed in detail.

The date-time formatting can be done in R in a variety of ways which can be explored by using the function as written below.

	?strftime

While the need of using so many formats may not be clearly visible to users, however, one of the factors which make it important in context of environmental data is that the environmental data is collected from different instruments which have their own data formats. So the availability of so many default formats in R makes it easy to work with date-time data from different sources. The way in which the function reads the date-time in the format provided by us has the same explanation as above.  
Some of the date-time formats are shown in the following examples in addition to the those already mentioned above.

	as.POSIXct("080406 10:11", format = "%y%m%d %H:%M");
	as.POSIXct("2008-04-06 10:11:01 PM", format = "%Y-%m-%d %I:%M:%S %p");
	as.POSIXct("08/04/06 22:11:00", format = "%m/%d/%y %H:%M:%S");

In the above examples, when the **AM/PM** has to be specified, then **%I** is used instead of **%H** and **%p** is used to indicate the **AM/PM**. Also, while **%m** is used for **months**, **%M** is used for **minutes**.  
Once the date-time has been read in the specified format, it can be changed to other format too as required using the functions **format()** or **as.character()** as shown

	format(as.POSIXct("080406 10:11", format = "%y%m%d %H:%M"), "%m/%d/%Y %I:%M %p");
	as.character(as.POSIXct("080406 10:11", format = "%y%m%d %H:%M"), format = "%m-%d-%y %H:%M");


##5. Behavior of "POSIXct" and "POSIXlt" classes within dataframes

The behavior of "POSIXct" and "POSIXlt" in dataframes is not simple. For instance, consider the following example.
  
**i. Create a dataframe**

	df <- data.frame(
			day = c(rep("20081101",5), rep("20081102",4), "20081103"), 
			time = c("01:20:00", "06:00:00", "12:20:00","17:30:00", "21:45:00", "01:15:00", "06:30:00", "12:50:00","20:00:00", "01:05:00"),
			value = c("5","5", "6", "6","5","5", "6", "7", "5", "5")
		  )

**ii. Add POSIXct and POSIXlt columns to dataframe**  
Combine the day and time columns from df and convert them to POSIXct and POSIXlt dataframes and merge again with df. Now an additional know-how is that the documentation recommends to use POSIXct when one has to work with dataframes
	
	df1 <- paste(df$day, df$time)
	df2 <- as.POSIXct(df1, format = "%Y%m%d %H:%M:%S")
	df3 <- as.POSIXlt(df1, format = "%Y%m%d %H:%M:%S")
	df.all <- data.frame(df, ct = df2, lt = df3)

**iii. Look at the structure of final dataframe**  
Now look at the structure of df.all. We observe that while "ct" was already of POSIXct category but "lt" which should have been of POSIXlt class is also coerced to POSIXct and that too without any warning.  

	str(df.all)

**iv. Using $ operator to repeat the same**  
Try to do the same thing using "$" operator and check the structure again. This time both "ct" and "lt" retain their class. It can also be checked by using the **unclass()** function

	df.all <- df
	df.all$ct <- df2
	df.all$lt <- df3
	str(df.all)
	unclass(df.all$ct)
	unclass(df.all$lt)

*The $ operator makes it work. Consider some more examples as below*  

**v. Lets try to round our data: "ct" column**  
When we try to round the time data stored in *ct* column of df.all, we observe that it parses the column to POSIXlt and gives a warning. This particular warning indicates that the parsing has failed. So we try to put back the data to the original state i.e. in POSIXct class.

	df.all[,"ct"] <- round(df.all[,"ct"], units= "hours")
	
	#Warning: provided 9 variables to replace 1 variable

	df.all[,"ct"] <- as.POSIXct(round(df.all[,"ct"], units= "hours"))

**vi. Rounding data: "lt" column**  
Try rounding the "lt" column of the df.all dataframe. It again gives the similar error as in case of "ct".

	df.all[,"lt"] <- round(df.all[,"lt"], units= "hours")
	
	#Warning: provided 9 variables to replace 1 variable

**vii. Rounding data using "$" operator**  
The above processes are tried using the $ operator. This time there is no warning and the code runs.

	df.all$ct <- round(df.all$ct, units= "hours")
	df.all$lt <- round(df.all$lt, units= "hours")

**viii. Try some other operations with and without using $ operator**
If we try to change only one element of the dataframe, then it gives an error if *$* is not used.

	df.all[1, "lt"] <- as.POSIXlt("2008-11-01 01:00:00")
	
	# Warning: provided 9 variables to replace 1 variables
	## Error: ’origin’ must be supplied

	df.all$lt[1] <- as.POSIXlt("2008-11-01 01:00:00")

	df.all[1, "lt"] <- as.POSIXct("2008-11-01 01:00:00")







## Ex.1 When do I turn one billion years old?

	bday <- as.character(readline(prompt= "Enter your birth date in the format 'dd-mm-yyyy-HH-MM-SS':  "))
	
	billday <- function(x, age= 10^9){
	  b.date <- as.POSIXct(x, format= "%d-%m-%Y-%H-%M-%S")
	  p.age <- round(difftime(Sys.time(), b.date, units= "days"))
	  r.date <- round(difftime(b.date + age, Sys.time(), units = "days"))
	  msg1 <- sprintf("Your present age is %s days", p.age)  
	  if(r.date > 0){
    	msg2 <- sprintf("You will be %s old on %s which is %s days from now", age, format(b.date + age, "%d-%m-%Y"), r.date)
	  } else {
    	msg2 <- sprintf("You turned %s old on %s which was %s days ago", age, format(b.date + age, "%d-%m-%Y"), -1*r.date)
	  }
		print(msg1)
		print(msg2)
	}
	
	billday(x= bday)


## Ex.2 When do I turn one billion years old?

	bday <- as.character(readline(prompt= "Enter your birth date in the format 'dd-mm-yyyy-HH-MM-SS':  "))
	
	billday <- function(x, age= 10^9){
	  b.date <- as.POSIXct(x, format= "%d-%m-%Y-%H-%M-%S")
	  p.age <- round(difftime(Sys.time(), b.date, units= "days"))
	  r.date <- round(difftime(b.date + age, Sys.time(), units = "days"))
	  msg1 <- sprintf("Your present age is %s days", p.age)  
	  if(r.date > 0){
    	msg2 <- sprintf("You will be %s old on %s which is %s days from now", age, format(b.date + age, "%d-%m-%Y"), r.date)
	  } else {
    	msg2 <- sprintf("You turned %s old on %s which was %s days ago", age, format(b.date + age, "%d-%m-%Y"), -1*r.date)
	  }
		print(msg1)
		print(msg2)
	}
	
	billday(x= bday)
