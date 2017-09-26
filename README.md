# Titanic-Suvival-Analysis
SAS code
libname s5238 "/courses/d0f434e5ba27fe300/sta5238";

/*keep the variables that may be used*/
data full;
set s5238.titanic_train(keep=passenger_class survived sex age siblings_and_spouses parents_and_children);
run;
proc means data=full;
run;

/*imputation*/
proc stdize data=full
out=impute_train
method=median
reponly;
var age;
run;
proc means data=impute_train;
var age;
run;

/*change sex into 0-1 form
(create a new variable "male" for male=1 and female=0, and drop sex)*/
data male(drop=sex);
set impute_train;
if sex in ('male') then do;
male=1;
end;
if sex in ('female') then do;
male=0;
end;
label male='male=1,female=0';
run;
proc means data=male n nmiss;
run;

/*new variable fs(family_size):add siblings_and_spouses and parents_and_children*/
data fs(drop=siblings_and_spouses parents_and_children);
set male;
fs=siblings_and_spouses+parents_and_children;
run;
proc means data=fs N Nmiss MAX Min;
run;

/*build model*/
proc logistic data=fs plots(only)=roc;
model survived(event="1")=passenger_class age male fs;
run;

/*Similar dealing with dataset titanic_test*/
data full_test;
set s5238.titanic_test(keep=passenger_class survived sex age siblings_and_spouses parents_and_children id);
run;
data male_test(drop=sex);
set full_test;
if sex in ('male') then do;
male=1;
end;
if sex in ('female') then do;
male=0;
end;
label male='male=1,female=0';
run;

/*imputation*/
proc stdize data=male_test
out=impute
method=median
reponly;
var age;
run;
data fs_test(drop=siblings_and_spouses parents_and_children);
set impute;
fs=siblings_and_spouses+parents_and_children;
run;
proc means data=fs_test N Nmiss MAX Min;
run;

/*assign survive*/
data titanic_t;
set fs_test;
decide=4.8005-1.1186*passenger_class-2.6243*male-0.0359*age-0.1788*fs;
if decide<0.25 then do;
survived=0;
end;
if decide>=0.25 then do;
survived=1;
end;
run;
proc print data=titanic_t;
run;

/*merge result with original dataset*/
data t_result(keep=survived id);
set titanic_t;
run;
data ti_test(drop=survived);
set s5238.titanic_test;
run;
data titanic_survived;
merge t_result ti_test;
by id;
run;
proc print data=titanic_survived;
run;
