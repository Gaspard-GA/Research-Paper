[Research_Paper_Stata_Outputs.log](https://github.com/user-attachments/files/19353663/Research_Paper_Stata_Outputs.log)#DOFILE

[cd "C:\Users\gaspa\OneDrive\Desktop\ECON 182\Research Project\Califronia Data"

import excel "Laborforce Annual Data"

rename (A B C D E F	G) (county_name	area_type year labor_force employment unemployment unemployment_rate)

drop if area_type != "County"

destring year, replace
drop if year >= 1990 & year <= 2011
sort county_name year

/*Merging "Stata dofile for ED Visits" into Stata dofile for Laborforce*/

merge 1:1 county_name year using "ED_Visits Data Per County Per Year"

drop if _merge==1

save "Cleaned_data of Labor and ED Visit Stats", replace

/*graph*/

sort county_name year

egen county= group(county_name)

xtset county year

destring labor_force unemployment, replace

/* TWOWAY TSLINE GRAPH*/
bysort year: egen count_avg = mean(count)
bysort year: egen labor_force_avg = mean(labor_force)
bysort year: egen unemployment_avg = mean(unemployment)

/* county level avg ploted agenst year*/
twoway line (count_avg unemployment_avg) year, sort xlabel(2012(2)2024, angle(360)) ylabel(, angle(0)) xtitle("Year") ytitle("Average ED Visits") title("Average Emergency Dapartment Visits Over Time")

graph export "Avg ED Visits Over Time in Relation to Unemployment.png", replace as(png)

/*Scatter Plot Graph*/
gen count_10k = count / 10000

twoway (scatter count_10k unemployment), xtitle("Unemployment") ytitle("ED Visits by 10k") title("Emergency Department Visits in 10K Vs. Unemployment (2012-2023)", span)

graph export "Unmeployment Vs. ED Visits (per 10k).png", replace as(png)

/*rescaling the graph devide unemployment by population and ED visits by population*/

import excel "California Population data", clear firstrow

drop COUNTY

drop L

reshape long Y, i(county_name)

rename (_j Y) (year population)

graph export "Population data.png", replace as(png)

merge 1:1 county_name year using "Cleaned_data of Labor and ED Visit Stats", nogen

drop if area_type==""
drop H I J
 
destring unemployment, replace force
gen upop = unemployment / population
gen cpop = count / population

/* scatter plot using ratios*/
encode county_name, generate(county_id)

ssc install palettes, replace
ssc install colrspace, replace

colorpalette tableau, n(48) nograph
local colors `r(colors)'

local plot_cmd ""
local legend_cmd ""

forvalues i = 1/48 {
    local color : word `i' of `colors'
    
    local plot_cmd `plot_cmd' (scatter cpop upop if county_id == `i', mcolor("`color'") msymbol(circle_small) msize(vsmall))
    local legend_cmd `legend_cmd' label(`i' "`:label county_id `i''")
}

twoway `plot_cmd', legend(order(1/48) `legend_cmd' size(tiny) rows(16)) xtitle("Unemployment by Population") ytitle("ED Visits by Population") title("Emergency Department Visits by County Vs. Unemployment (2012-2023)", span)

graph export "Twoway Unemployment Vs. ED Visits by County.png", replace as(png)

/* reggression of ED on unemplyment with the county fixed effects*/
ssc install reghdfe
ssc install ftools

label variable cpop "ED Visits per Capita"
label variable upop "Unemployment per Capita"
/*regression with Fixed effect and without*/
reg cpop upop, vce(robust)
outreg2 using results.tex, tex(frag) label bdec(3) replace

reghdfe cpop upop, absorb(county_name) vce(robust)
outreg2 using results.tex, tex(frag) label append addtext(County FE, Yes)

/* regression of upop on cpop for LA county*/
reghdfe cpop upop if county_name == "Los Angeles County", vce(robust)
outreg2 using results2.tex, tex(frag) label replace bdec(3)

/* regression of upop on cpop for Fresno County (#1 in agrecultural farming)*/
reghdfe cpop upop if county_name == "Fresno County", vce(robust)
outreg2 using results2.tex, tex(frag) label append bdec(3)

/*regression of upop on cpop for Tulare County (#1 in cattle farming)*/
reghdfe cpop upop if county_name == "Tulare County", vce(robust)
outreg2 using results2.tex, tex(frag) append label bdec(3)

/* comparasent 2 way graph of the three counties above*/
twoway (scatter cpop upop if county_name=="Los Angeles County", mcolor(blue)) (lfit cpop upop if county_name=="Los Angeles County", lcolor(blue)) (scatter cpop upop if county_name=="Fresno County", mcolor(red)) (lfit cpop upop if county_name=="Fresno County", lcolor(red)) (scatter cpop upop if county_name=="Tulare County", mcolor(green)) (lfit cpop upop if county_name=="Tulare County", lcolor(green)), legend(order(1 "Los Angeles County" 3 "Fresno County" 5 "Tulare County")) ytitle("Emergency Department Visits Rate") xtitle("Unemployment Rate") title("Industry Effects on ED Visits Vs. Unemployment in 3 Counties (2012-2023)", span)



[----------------------------------------------------------------------------------------------------------------------------------------------
      name:  <unnamed>
       log:  C:\Users\gaspa\OneDrive\Desktop\ECON 182\Research Project\Califronia Data\Research_Paper_Stata_Outputs.log
  log type:  text
 opened on:  19 Mar 2025, 16:16:56

. 
. import excel "Laborforce Annual Data"
(10 vars, 16,054 obs)

. 
. rename (A B C D E F     G) (county_name area_type year labor_force employment unemployment unemployment_rate)

. 
. drop if area_type != "County"
(14,082 observations deleted)

. 
. destring year, replace
year: all characters numeric; replaced as int

. drop if year >= 1990 & year <= 2011
(1,276 observations deleted)

. sort county_name year

. 
. /*Merging "Stata dofile for ED Visits" into Stata dofile for Laborforce*/
. 
. merge 1:1 county_name year using "ED_Visits Data Per County Per Year"

    Result                      Number of obs
    -----------------------------------------
    Not matched                           120
        from master                       120  (_merge==1)
        from using                          0  (_merge==2)

    Matched                               576  (_merge==3)
    -----------------------------------------

. 
. drop if _merge==1
(120 observations deleted)

. 
. save "Cleaned_data of Labor and ED Visit Stats", replace
file Cleaned_data of Labor and ED Visit Stats.dta saved

. 
. /*graph*/
. 
. sort county_name year

. 
. egen county= group(county_name)

. 
. xtset county year

Panel variable: county (strongly balanced)
 Time variable: year, 2012 to 2023
         Delta: 1 unit

. 
. destring labor_force unemployment, replace
labor_force: all characters numeric; replaced as long
unemployment: all characters numeric; replaced as long

. 
. /* TWOWAY TSLINE GRAPH*/
. bysort year: egen count_avg = mean(count)

. bysort year: egen labor_force_avg = mean(labor_force)

. bysort year: egen unemployment_avg = mean(unemployment)

. 
. /* county level avg ploted agenst year*/
. twoway line (count_avg unemployment_avg) year, sort xlabel(2012(2)2024, angle(360)) ylabel(, angle(0)) xtitle("Year") ytitle("Average ED Vis
> its") title("Average Emergency Dapartment Visits Over Time")

. 
. graph export "Avg ED Visits Over Time in Relation to Unemployment.png", replace as(png)
file Avg ED Visits Over Time in Relation to Unemployment.png saved as PNG format

. 
. /*Scatter Plot Graph*/
. gen count_10k = count / 10000

. 
. twoway (scatter count_10k unemployment), xtitle("Unemployment") ytitle("ED Visits by 10k") title("Emergency Department Visits in 10K Vs. Une
> mployment (2012-2023)", span)

. 
. graph export "Unmeployment Vs. ED Visits (per 10k).png", replace as(png)
file Unmeployment Vs. ED Visits (per 10k).png saved as PNG format

. 
. /*rescaling the graph devide unemployment by population and ED visits by population*/
. 
. import excel "California Population data", clear firstrow
(15 vars, 58 obs)

. 
. drop COUNTY

. 
. drop L

. 
. reshape long Y, i(county_name)
(j = 2012 2013 2014 2015 2016 2017 2018 2019 2020 2021 2022 2023)

Data                               Wide   ->   Long
-----------------------------------------------------------------------------
Number of observations               58   ->   696         
Number of variables                  13   ->   3           
j variable (12 values)                    ->   _j
xij variables:
                  Y2012 Y2013 ... Y2023   ->   Y
-----------------------------------------------------------------------------

. 
. rename (_j Y) (year population)

. 
. graph export "Population data.png", replace as(png)
file Population data.png saved as PNG format

. 
. merge 1:1 county_name year using "Cleaned_data of Labor and ED Visit Stats", nogen
(variable county_name was str22, now str49 to accommodate using data's values)

    Result                      Number of obs
    -----------------------------------------
    Not matched                           120
        from master                       120  
        from using                          0  

    Matched                               576  
    -----------------------------------------

. 
. drop if area_type==""
(120 observations deleted)

. drop H I J

.  
. destring unemployment, replace force
unemployment: all characters numeric; replaced as long

. gen upop = unemployment / population

. gen cpop = count / population

. 
. /* scatter plot using ratios*/
. encode county_name, generate(county_id)

. 
. ssc install palettes, replace
checking palettes consistency and verifying not already installed...
all files already exist and are up to date.

. ssc install colrspace, replace
checking colrspace consistency and verifying not already installed...
all files already exist and are up to date.

. 
. colorpalette tableau, n(48) nograph

. local colors `r(colors)'

. 
. local plot_cmd ""

. local legend_cmd ""

. 
. forvalues i = 1/48 {
  2.     local color : word `i' of `colors'
  3.     
.     local plot_cmd `plot_cmd' (scatter cpop upop if county_id == `i', mcolor("`color'") msymbol(circle_small) msize(vsmall))
  4.     local legend_cmd `legend_cmd' label(`i' "`:label county_id `i''")
  5. }

. 
. twoway `plot_cmd', legend(order(1/48) `legend_cmd' size(tiny) rows(16)) xtitle("Unemployment by Population") ytitle("ED Visits by Population
> ") title("Emergency Department Visits by County Vs. Unemployment (2012-2023)", span)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
(note:  named style circle_small not found in class symbol, default attributes used)
1/48 not an integer, option order() ignored

. 
. graph export "Twoway Unemployment Vs. ED Visits by County.png", replace as(png)
file Twoway Unemployment Vs. ED Visits by County.png saved as PNG format

. 
. /* reggression of ED on unemplyment with the county fixed effects*/
. ssc install reghdfe
checking reghdfe consistency and verifying not already installed...
all files already exist and are up to date.

. ssc install ftools
checking ftools consistency and verifying not already installed...
all files already exist and are up to date.

. 
. label variable cpop "ED Visits per Capita"

. label variable upop "Unemployment per Capita"

. /*regression with Fixed effect and without*/
. reg cpop upop, vce(robust)

Linear regression                               Number of obs     =        576
                                                F(1, 574)         =       0.65
                                                Prob > F          =     0.4220
                                                R-squared         =     0.0008
                                                Root MSE          =     .14485

------------------------------------------------------------------------------
             |               Robust
        cpop | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
        upop |   .2759347      .3434     0.80   0.422    -.3985391    .9504086
       _cons |   .3450966   .0121941    28.30   0.000     .3211461    .3690471
------------------------------------------------------------------------------

. outreg2 using results.tex, tex(frag) label bdec(3) replace
results.tex
dir : seeout

. 
. reghdfe cpop upop, absorb(county_name) vce(robust)
(MWFE estimator converged in 1 iterations)

HDFE Linear regression                            Number of obs   =        576
Absorbing 1 HDFE group                            F(   1,    527) =     103.42
                                                  Prob > F        =     0.0000
                                                  R-squared       =     0.9235
                                                  Adj R-squared   =     0.9166
                                                  Within R-sq.    =     0.1516
                                                  Root MSE        =     0.0418

------------------------------------------------------------------------------
             |               Robust
        cpop | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
        upop |  -1.557332   .1531358   -10.17   0.000    -1.858164   -1.256501
       _cons |   .4048326   .0052302    77.40   0.000      .394558    .4151073
------------------------------------------------------------------------------

Absorbed degrees of freedom:
-----------------------------------------------------+
 Absorbed FE | Categories  - Redundant  = Num. Coefs |
-------------+---------------------------------------|
 county_name |        48           0          48     |
-----------------------------------------------------+

. outreg2 using results.tex, tex(frag) label append addtext(County FE, Yes)
results.tex
dir : seeout

. 
. /* regression of upop on cpop for LA county*/
. reghdfe cpop upop if county_name == "Los Angeles County", vce(robust)
(MWFE estimator converged in 1 iterations)

HDFE Linear regression                            Number of obs   =         12
Absorbing 1 HDFE group                            F(   1,     10) =     177.12
                                                  Prob > F        =     0.0000
                                                  R-squared       =     0.9407
                                                  Adj R-squared   =     0.9348
                                                  Within R-sq.    =     0.9407
                                                  Root MSE        =     0.0067

------------------------------------------------------------------------------
             |               Robust
        cpop | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
        upop |  -1.893803   .1422989   -13.31   0.000    -2.210865   -1.576742
       _cons |   .3408856   .0061784    55.17   0.000     .3271192    .3546519
------------------------------------------------------------------------------

. outreg2 using results2.tex, tex(frag) label replace bdec(3)
results2.tex
dir : seeout

. 
. /* regression of upop on cpop for Fresno County (#1 in agrecultural farming)*/
. reghdfe cpop upop if county_name == "Fresno County", vce(robust)
(MWFE estimator converged in 1 iterations)

HDFE Linear regression                            Number of obs   =         12
Absorbing 1 HDFE group                            F(   1,     10) =      73.86
                                                  Prob > F        =     0.0000
                                                  R-squared       =     0.7763
                                                  Adj R-squared   =     0.7540
                                                  Within R-sq.    =     0.7763
                                                  Root MSE        =     0.0090

------------------------------------------------------------------------------
             |               Robust
        cpop | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
        upop |  -1.257503    .146321    -8.59   0.000    -1.583527   -.9314797
       _cons |   .2101631   .0082623    25.44   0.000     .1917536    .2285725
------------------------------------------------------------------------------

. outreg2 using results2.tex, tex(frag) label append bdec(3)
results2.tex
dir : seeout

. 
. /*regression of upop on cpop for Tulare County (#1 in cattle farming)*/
. reghdfe cpop upop if county_name == "Tulare County", vce(robust)
(MWFE estimator converged in 1 iterations)

HDFE Linear regression                            Number of obs   =         12
Absorbing 1 HDFE group                            F(   1,     10) =       2.47
                                                  Prob > F        =     0.1469
                                                  R-squared       =     0.0925
                                                  Adj R-squared   =     0.0018
                                                  Within R-sq.    =     0.0925
                                                  Root MSE        =     0.0343

------------------------------------------------------------------------------
             |               Robust
        cpop | Coefficient  std. err.      t    P>|t|     [95% conf. interval]
-------------+----------------------------------------------------------------
        upop |   .9868925   .6274976     1.57   0.147    -.4112593    2.385044
       _cons |     .22091   .0329674     6.70   0.000      .147454    .2943659
------------------------------------------------------------------------------

. outreg2 using results2.tex, tex(frag) append label bdec(3)
results2.tex
dir : seeout

. 
. /* comparasent 2 way graph of the three counties above*/
. twoway (scatter cpop upop if county_name=="Los Angeles County", mcolor(blue)) (lfit cpop upop if county_name=="Los Angeles County", lcolor(b
> lue)) (scatter cpop upop if county_name=="Fresno County", mcolor(red)) (lfit cpop upop if county_name=="Fresno County", lcolor(red)) (scatte
> r cpop upop if county_name=="Tulare County", mcolor(green)) (lfit cpop upop if county_name=="Tulare County", lcolor(green)), legend(order(1 
> "Los Angeles County" 3 "Fresno County" 5 "Tulare County")) ytitle("Emergency Department Visits Rate") xtitle("Unemployment Rate") title("Ind
> ustry Effects on ED Visits Vs. Unemployment in 3 Counties (2012-2023)", span)

. 
. graph export "Twoway Comparasent Graph of Three Counties.png", replace as(png)
file Twoway Comparasent Graph of Three Counties.png saved as PNG format

. 
. log close
      name:  <unnamed>
       log:  C:\Users\gaspa\OneDrive\Desktop\ECON 182\Research Project\Califronia Data\Research_Paper_Stata_Outputs.log
  log type:  text
 closed on:  19 Mar 2025, 16:17:33
----------------------------------------------------------------------------------------------------------------------------------------------
Uploading Research_Paper_Stata_Outputs.log…]()

graph export "Twoway Comparasent Graph of Three Counties.png", replace as(png)Uploading Stata dofile for Laborforce.do…]()
