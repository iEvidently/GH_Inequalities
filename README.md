# Global Health Module [MD999]
## Measuring Inequalities

![Course logo](/images/md999.png)
## Systema Set-up

### Install Stata - Software Centre

![Software Centre 1](/images/computer1.png)

![Software Centre 2](/images/computer2.png)

### Install additional commands

![Stata](/images/stata.png)

```
ssc install conindex
ssc install oaxaca
```

### Additional set-up

```
set more off
set scheme economist
log using analysis, replace text
```

## Load data

### From Excel sheet
```
insheet using https://s3.amazonaws.com/ghinequality/gh_data.csv, clear

```

### From native Stata file
```
use https://s3.amazonaws.com/ghinequality/gh_data.dta, clear

```

## Explore that the data

```
codebook

describe 



```

## Generate outcome variable

**Stunting in a nutshell**

Stunting is the impaired growth and development that children experience from poor nutrition, repeated infection, and inadequate psychosocial stimulation. Children are defined as *stunted if their height-for-age is more than two standard deviations* below the WHO Child Growth Standards median.

```
hist haz

gen stunt = haz < - 2
```

## 1. Concept of Disparity

Is there a difference in stunting by maternal educational attainment?

```
tab medu stunt
tab medu stunt, row
tab medu stunt, row chi
```

Graphically ...

```
graph hbar stunt, over(medu) ytitle("Proportion Stunted")
```

Relative measure:

```
logistic stunt i.medu
tab medu, nolabel
logistic stunt b2.medu
````


## 2. Concept of Inequity

Are the ethnic differences in childhood stunting rates that we observe attributable to education difference?

![Map](/images/map2.png)
```
gen noed = (medu ==0)
tab region noed, row chi
graph hbar  noed, over(region) ytitle("Proportion with No education")
tab region, nolabel
```

Generate a new variable for North region
```
recode region (2 3 = 1 "North") (1 4 5 6 = 0 "Others"), gen(north)
```

Is there evidence of difference
```
cs stunt noed 
```


## 3. Concept of Inequality

To check for degree of association betwen difference in rates of childhood stunting between wealth groups

```
tab wealth stunt, row chi
graph hbar stunt, over(wealth) ytitle("Proportion Stunted")
tab wealth, nolabel

logistic stunt b5.wealth
```

Now, calculate the Concentration Index

```
conindex stunt, rankvar(wealth) generalized truezero 
```


## 4. Concept of Burden

Which wealth group has the greatest burden?

Check wealth the group is fairly distributed
```
tab wealth
```

Greatest burden:

```
tab wealth stunt
```

Graphically:

```
graph hbar (count) stunt, over(wealth) ytitle("Number of Stunted Children")
```

## 5. Blinder Oaxaca decomposition analysis
To determine the magnitude in rural-urban disparities in childhood malnutrition and decompose the gap:

Magnitude
```
tab rural stunt, row chi
cs stunt rural
```

Decomposition Analysis

```
oaxaca stunt region wealth mage medu  media com_zhser5 com_zsesc5 , by(rural)   logit   pooled
```

What are the important factors:

```
local names : colfullnames e(b)
matrix k = e(b)
svmat k, name(var)


preserve

keep var*

gen id = _n
reshape long var, i(id) j(cat)
keep if !missing(var)

gen str factorLabel = ""
local j "1"
foreach name of local names {
	di "`name'"
	di "`j'"
	replace factorLabel = "`name'" in `j'
	local ++j
}



gen perc = (var / var[3])*100

keep in 6/12


graph hbar perc, over(factorLabel, sort(perc) descend) blabel(bar, position(inside) format(%9.1f) color(white))  ytitle("Percentage Stunted") sort(perc)  				  


```








