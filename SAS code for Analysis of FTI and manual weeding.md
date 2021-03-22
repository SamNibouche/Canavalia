## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.TFI    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'FTI and manual weeding.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
       informat locality $12. ;
       informat crop_cycle $5. ;
       informat treatment $8. ;
       informat TFI best32. ;
       informat manual_weeding best32. ;
       format locality $12. ;
       format crop_cycle $5. ;
       format treatment $8. ;
       format TFI best12. ;
       format manual_weeding best12. ;
    input
                locality $
                crop_cycle $
                treatment $
                TFI
                manual_weeding
    ;
    if _ERROR_ then call symputx('_EFIERR_',1);  /* set ERROR detection macro variable */
run;
```
### Data analysis
#### Table 3: statistical analysis
```
proc glm data=TFI;
	class treatment locality crop_cycle;
	model TFI = treatment locality(crop_cycle);
	means treatment/ snk;
	lsmeans treatment/pdiff adjust=tukey;
run; quit;

proc glm data=TFI;
	class treatment locality crop_cycle;
	model manual_weeding = treatment locality(crop_cycle);
	lsmeans treatment/pdiff adjust=tukey;
run; quit;
```
