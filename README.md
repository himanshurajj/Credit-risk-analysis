# Credit-risk-analysis

LIBNAME credit "/home/u61854492/Credit";
RUN;
Data example;
SET credit.mortgage;
LIBNAME Data "/home/u61854492/Credit";
RUN;
Data example;
SET data.mortgage;

/*Example for deletion of observations*/
IF FICO_orig_time< 500 THEN DELETE;
/*Example for generation of new variables*/
IF FICO_orig_time> 500 THEN FICO_cat=1;
IF FICO_orig_time> 700 THEN FICO_cat=2;
/*Example for data filtering*/
WHERE default_time=1;
/*Example for dropping of variables*/
DROP status_time;
RUN;
/*Means computes the mean, standard deviation, minimum, and maximum for the
variables default_time, FICO_orig_time, ltv_orig_time, and gdp_time of the mortgage
data set*/
PROC MEANS DATA=data.mortgage;
VAR default_time FICO_orig_time ltv_orig_time gdp_time;
RUN;
/* liner reg*/
PROC REG DATA=data.mortgage;
MODEL default_time = FICO_orig_time ltv_orig_time gdp_time;
RUN;

/*Macros
commence with the %MACRO command and conclude with the %MEND. Themerit
of the following macro is that you don’t have to replicate PROC REG*/
%MACRO example(datain, lhs, rhs);
PROC REG DATA=&datain;
MODEL &lhs = &rhs;
RUN;
%MEND example;

%example(datain=data.mortgage, lhs=default_time,
rhs=FICO_orig_time );
%example(datain=data.mortgage, lhs=default_time,
rhs=FICO_orig_time ltv_orig_time);
%example(datain=data.mortgage, lhs=default_time,
rhs=FICO_orig_time ltv_orig_time gdp_time);

/*export data8*/

PROC EXPORT exports these parameter
estimates into a CSV file in the specified location path ‘C:\Users\himanshu.raj\OneDrive - Xceedance Consulting India Private Ltd\Desktop\Study\SAS\export.csv’.
ODS LISTING CLOSE;
ODS OUTPUT PARAMETERESTIMATES=parameters;
PROC REG DATA=DATA.mortgage;
MODEL default_time = FICO_orig_time ltv_orig_time gdp_time;
RUN;
ODS OUTPUT CLOSE;
ODS LISTING;
PROC EXPORT DATA=parameters REPLACE DBMS=CSV OUTFILE=“C:\Users\himanshu.raj\OneDrive - Xceedance Consulting India Private Ltd\Desktop\Study\SAS\export.csv”;
RUN;
/* one dimension analysis*/

/* to find frequency*/ 
PROC FREQ DATA=data.mortgage;
TABLES default_time;
RUN;

ODS GRAPHICS ON;
PROC UNIVARIATE DATA=data.mortgage;
VAR FICO_orig_time LTV_orig_time;
CDFPLOT FICO_orig_time LTV_orig_time;
HISTOGRAM FICO_orig_time LTV_orig_time;
RUN;
ODS GRAPHICS OFF;

/* locations measures by mean median mode quantiles*/
PROC MEANS DATA=data.mortgage
N MEAN MEDIAN MODE P1 P99 MAXDEC=4;
VAR default_time FICO_orig_time LTV_orig_time;
RUN;

/*noramal distribution qq plots for seeing emperical and theoritical data should be same*/
ODS GRAPHICS ON;
PROC UNIVARIATE DATA=data.mortgage NOPRINT;
QQPLOT FICO_orig_time LTV_orig_time
/NORMAL(MU=EST SIGMA=EST COLOR=LTGREY) ;
RUN;
ODS GRAPHICS OFF;

/*to find the dispersion measures like range ,mse, variance, sd,cv*/
PROC MEANS DATA=data.mortgage
N MIN MAX RANGE QRANGE VAR STD CV MAXDEC=4;
VAR default_time FICO_orig_time LTV_orig_time;
RUN;

/* to find skewness and kurtosis*/
PROC MEANS DATA=data.mortgage
N SKEW KURT MAXDEC=4;
VAR default_time FICO_orig_time LTV_orig_time;
RUN;

/* two dimensional analysis*/

DATA mortgage1;
SET data.mortgage;
RUN;
PROC SORT DATA = mortgage1;
BY id time;
RUN;
PROC RANK DATA = mortgage1
GROUPS=5
OUT=quint(KEEP=id time FICO_orig_time);
VAR FICO_orig_time;
RUN;
DATA new;
MERGE mortgage1 quint;
BY id time;
RUN;
PROC FREQ DATA=new ;
TABLES default_time*FICO_ORIG_TIME;
RUN;

/*BOX plot -to measure the interrelation */
PROC SORT DATA= mortgage1;
BY default_time;
RUN;
ODS GRAPHICS ON;
PROC BOXPLOT DATA = mortgage1;
PLOT FICO_orig_time*default_time /IDSYMBOL=CIRCLE
IDHEIGHT=2 CBOXES=BLACK BOXWIDTH=10 ;
RUN;
ODS GRAPHICS OFF;
ODS GRAPHICS ON;
PROC BOXPLOT DATA = mortgage1;
PLOT LTV_orig_time*default_time / IDSYMBOL=CIRCLE
IDHEIGHT=2 CBOXES=BLACK BOXWIDTH=10 ;
RUN;
ODS GRAPHICS OFF;

/* measure dependance we use chi sq phi coeff*/
PROC FREQ DATA=new;
TABLES default_time*FICO_ORIG_TIME / CHISQ;
RUN;

/* measure correlation*/
DATA sample;
SET data.mortgage;
IF RANUNI(123456) < 0.01;
RUN;
ODS GRAPHICS ON;
PROC CORR DATA=sample
PLOTS(MAXPOINTS=NONE)=SCATTER(NVAR=2 ALPHA=.20 .30)
KENDALL SPEARMAN;
VAR FICO_orig_time LTV_orig_time;
RUN;
ODS GRAPHICS OFF;

/* to find the confidance interval*/

ODS SELECT BASICINTERVALS;
PROC UNIVARIATE DATA=data.mortgage CIBASIC(ALPHA=.01);
VAR LTV_orig_time;
RUN;

/*hypothesis testing*/

ODS GRAPHICS ON;
ODS SELECT TESTSFORLOCATION ;
PROC UNIVARIATE DATA=data.mortgage MU0=60;
VAR LTV_orig_time;
RUN;
ODS GRAPHICS OFF;

/*sampaling of data ---- The options of PROC SURVEYSELECT can be
explained as follows:
◾ METHOD=SRS: Applies simple random sampling.
◾ N=1000: Creates a sample of 1,000 observations.
◾ SEED=12345: The seed needed to randomly sample the observations.
◾ OUT=data.mySample: The result will be stored in the data set mySample.
◾ STRATA bad / ALLOC=PROP: The stratification variable is bad and the allocation
method used is proportional.*/

PROC SORT DATA=data.hmeq;
BY bad;
RUN;
PROC SURVEYSELECT DATA=data.hmeq
METHOD=SRS N=1000 SEED=12345 OUT=data.mySample;
STRATA bad / ALLOC=PROP;
RUN;
PROC FREQ DATA=data.hmeq;
TABLES bad;
RUN;
PROC FREQ DATA=data.mySample;
TABLES bad;
RUN;

/** histogram for loan variable**/
PROC UNIVARIATE DATA=data.mySample;
VAR loan;
HISTOGRAM;
RUN;

/** pie chart for job variable**/
PROC GCHART DATA=data.mySample;
PIE job;
RUN;

/** scatter plot of loan versus
mortdue**/
PROC GPLOT DATA=data.mySample;
PLOT loan*mortdue;
RUN;
/*Descriptive statistics can be calculated in Base SAS using PROC UNIVARIATE as
follows:*/
PROC UNIVARIATE DATA=data.mySample;
VAR loan;
RUN;

/* In Base SAS, PROC STANDARD can be used to replace missing values. Consider
the following statement:*/
PROC STANDARD DATA =data.mysample REPLACE OUT=data.mysamplenomissing;
RUN;
/*This will replace allmissing values of the continuous variables with the mean. The
missing values for the categorical variables will remain untreated*/

/*In Base SAS, we can filter outliers based on the z-scores by using PROC STANDARD
as follows:*/

PROC STANDARD DATA =data.mysample MEAN=0 STD=1 OUT=data.zscores;
VAR clage clno debtinc delinq derog loan mortdue ninq value yoj;
RUN;

DATA data.filteredsample;
SET data.zscores;
WHERE ABS(clage) < 3 & ABS(clno) <3 & ABS(debtinc) < 3 & ABS(delinq)<3
& ABS(derog) < 3
& ABS(loan) < 3 & ABS(mortdue)< 3 & ABS(ninq)<3 & ABS(value) < 3 & ABS(yoj)
< 3;
RUN;

/*In Base SAS, we can start by first defining the data set as follows:*/
DATA residence;
INPUT default$ resstatus$ count;
DATALINES;
good owner 6000
good rentunf 1600
good rentfurn 350
good withpar 950
good other 90
good noanswer 10
bad owner 300
bad rentunf 400
bad rentfurn 140
bad withpar 100
bad other 50
bad noanswer 10
;
/* We can now also create the data sets for both categorization options:*/
DATA coarse1;
INPUT default$ resstatus$ count;
DATALINES;
good owner 6000
good renter 1950
good other 1050
bad owner 300
bad renter 540
bad other 160
;
DATA coarse2;
INPUT default$ resstatus$ count;
DATALINES;
good owner 6000
good withpar 950
good other 2050
bad owner 300
bad withpar 100
bad other 600
;
/* We can now run PROC FREQ on both options as follows:*/
PROC FREQ DATA=coarse1;
WEIGHT count;
TABLES default*resstatus / CHISQ;
RUN;
/*This will give the output depicted in Exhibit 4.29.
We can now also examine Option 2 as follows:*/
PROC FREQ DATA=coarse2;
WEIGHT count;
TABLES default*resstatus / CHISQ;
RUN;
/*This will give the output depicted in Exhibit 4.30.*/

/* logistic regression*/
PROC LOGISTIC DATA=data.hmeq;
CLASS job reason /PARAM=glm;
MODEL bad=clage clno debtinc delinq derog job loan mortdue ninq reason
value yoj/
SELECTION=stepwise SLENTRY=0.05 SLSTAY=0.01;
RUN;

