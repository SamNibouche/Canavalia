## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.GROUND_COVER    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'global ground cover.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
       informat notation_date ddmmyy10. ;
       informat crop_cycle $5. ;
       informat bloc best32. ;
       informat global_score_WP best32. ;
       informat global_score_IR best32. ;
       informat weed_score_WP best32. ;
       informat weed_score_IR best32. ;
       informat canavalia_score_IR best32. ;
       informat treatment $3. ;
       informat locality $10. ;
       format notation_date ddmmyy10. ;
       format crop_cycle $5. ;
       format bloc best12. ;
       format global_score_WP best12. ;
       format global_score_IR best12. ;
       format weed_score_WP best12. ;
       format weed_score_IR best12. ;
       format canavalia_score_IR best12. ;
       format treatment $3. ;
       format locality $10. ;
    input
                notation_date
                crop_cycle $
                bloc
                global_score_WP
                global_score_IR
                weed_score_WP
                weed_score_IR
                canavalia_score_IR
                treatment $
                locality $
    ;
    if _ERROR_ then call symputx('_EFIERR_',1);  /* set ERROR detection macro variable */
run;
```
### Data analysis
#### Table 3: statistical analysis
```
proc sort data=ground_cover; 
	by locality crop_cycle bloc treatment  ; 
run;
proc means data=ground_cover noprint;
	var global_score_WP global_score_IR weed_score_WP weed_score_IR canavalia_score_IR; 
	by locality crop_cycle bloc treatment;
	output out=mean_cover mean=;
run;

%macro tem(note);
ods output Diffs = cmm_&note ;
title "comparaison de &note. avec témoins sans effet date de semis des pds";
	proc mixed data=mean_cover covtest;
		class locality crop_cycle bloc treatment;
		model &note =  treatment bloc(locality) locality treatment*locality /outp=resid_&note ;
		repeated crop_cycle /subject=treatment*bloc(locality) group=locality type=CS;
		lsmeans treatment*locality/pdiff adjust=tukey slice=locality;
	run;
	data cmm_&note;
		set cmm_&note;
		where locality = _locality;
	run;
%mend;

%tem(weed_score_IR);
*%tem(global_score_WP);
*%tem(weed_score_WP);
*%tem(global_score_IR);
*%tem(canavalia_score_IR);
```
