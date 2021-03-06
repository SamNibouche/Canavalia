## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.yield    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'yield.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
       informat treatment $4. ;
       informat crop_cycle $5. ;
       informat plot $3. ;
       informat bloc best32. ;
       informat yield best32. ;
       informat locality $12. ;
       format treatment $4. ;
       format crop_cycle $5. ;
       format plot $3. ;
       format bloc best12. ;
       format yield best12. ;
       format locality $12. ;
    input
                treatment $
                crop_cycle $
                plot $
                bloc
                yield
                locality $
    ;
    if _ERROR_ then call symputx('_EFIERR_',1);  /* set ERROR detection macro variable */
run;
```
### Data analysis
#### Table 3: statistical analysis
```
ods output Diffs = cmm;
proc mixed data=yield covtest;
	class locality crop_cycle bloc plot treatment;
	model yield = treatment bloc(locality) locality treatment*locality/outp=resid s;
	repeated crop_cycle /subject=treatment*bloc(locality) group=locality type=vc;
	lsmeans treatment/pdiff adjust=tukey;
	lsmeans treatment*locality/ pdiff adjust=tukey slice=locality;
run;
```
#### Pairwised mean comparisons by locality
```
data cmm;
	set cmm;
	where locality = _locality;
run;
```
