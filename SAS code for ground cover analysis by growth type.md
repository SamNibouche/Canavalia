## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.species    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'ground cover by growth type & species.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
       informat notation_date mmddyy10. ;
       informat treatment $4. ;
       informat crop_cycle $5. ;
       informat bloc best32. ;
       informat plot $3. ;
       informat age best32. ;
       informat esp $5. ;
       informat cover best32. ;
       informat pres best32. ;
       informat growth_type $10. ;
       informat locality $12. ;
       format notation_date mmddyy10. ;
       format treatment $4. ;
       format crop_cycle $5. ;
       format bloc best12. ;
       format plot $3. ;
       format age best12. ;
       format esp $5. ;
       format cover best12. ;
       format pres best12. ;
       format growth_type $10. ;
       format locality $12. ;
    input
                notation_date
                treatment $
                crop_cycle $
                bloc
                plot $
                age
                esp $
                cover
                pres
                growth_type $
                locality $
    ;
    if _ERROR_ then call symputx('_EFIERR_',1);  /* set ERROR detection macro variable */
run;

proc sort data=species;
	by locality crop_cycle bloc treatment plot notation_date growth_type;
run;

proc means data=species noprint;
	by locality crop_cycle bloc treatment plot notation_date growth_type;
	var cover;
	output out = growth_type_date sum=;
run;

proc sort data=growth_type_date;
	by locality crop_cycle bloc treatment plot growth_type;
run;

proc means data=growth_type_date noprint;
	by locality crop_cycle bloc treatment plot growth_type;
	var cover;
	output out = growth_type_plot mean=cover_mean;
run;

proc sort data=growth_type_plot;
	by growth_type;
run;
```
### Data analysis
#### Table 3: statistical analysis
```
ods output Diffs = cmm;
proc mixed data=growth_type_plot covtest scoring=5 ;
	class locality crop_cycle bloc treatment;
	model cover_mean =  treatment bloc(locality*crop_cycle) locality treatment*locality/outp=resid s;
	repeated crop_cycle /subject=treatment*bloc*locality group=locality type=cs;
	lsmeans treatment/pdiff adjust=tukey ;
	lsmeans treatment*locality/slice=locality pdiff adjust=tukey;
	by growth_type;
run;
```
#### Pairwised mean comparisons
```
data cmm;
	set cmm;
	where locality = _locality;
run;
```
