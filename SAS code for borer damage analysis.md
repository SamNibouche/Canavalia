## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.borer    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'borer damage.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
       informat plot best32. ;
       informat stalk best32. ;
       informat ENT best32. ;
       informat ENA best32. ;
       informat bloc best32. ;
       informat ta best32. ;
       informat crop_cycle $5. ;
       informat treatment $3. ;
       informat locality $10. ;
       format plot best12. ;
       format stalk best12. ;
       format ENT best12. ;
       format ENA best12. ;
       format bloc best12. ;
       format ta best12. ;
       format crop_cycle $5. ;
       format treatment $3. ;
       format locality $10. ;
    input
                plot
                stalk
                ENT
                ENA
                bloc
                ta
                crop_cycle $
                treatment $
                locality $
    ;
    if _ERROR_ then call symputx('_EFIERR_',1);  /* set ERROR detection macro variable */
 run;

proc sort data=borer; 
  by locality crop_cycle treatment bloc plot ; 
run;

proc means data=borer noprint;
	by locality crop_cycle treatment bloc plot ;
	var ta;
	output out=borer_ta sum=ta n=nb_tig;
run;
```
### Data analysis
#### Table 3: statistical analysis
```
pods output Diffs = cmm;
proc glimmix data=borer_ta MAXOPT=100 PCONV=.000015 scoring=50;
	class bloc treatment crop_cycle locality bloc;
	model ta/nb_tig = treatment bloc(locality*crop_cycle) locality treatment*locality/dist=bin ;
	random crop_cycle /subject=treatment*bloc(locality) group=locality type=cs residual;
	lsmeans treatment*locality/pdiff ilink adjust=tukey  slicediff=locality;
run;
```
#### Pairwised mean comparisons by locality
```
data cmm;
	set cmm;
	where locality = _locality;
run;
```
