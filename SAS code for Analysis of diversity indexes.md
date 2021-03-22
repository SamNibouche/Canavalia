## Data analysis
### Data
Raw data are available at [DOI:10.18167/DVN1/NVW8ZV] (http://dx.doi.org/10.18167/DVN1/NVW8ZV)
### Import data
```
data WORK.species    ;
    %let _EFIERR_ = 0; /* set the ERROR detection macro variable */
    infile 'D:\Mes Données\Etudes\Push pull\Ecocanne\Travaux\Action 3\Dataverse\ground cover by growth type & species.csv' delimiter = ',' MISSOVER DSD lrecl=32767 firstobs=2 ;
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
```
### Computation of diversity indexes
## adapted from http://www.scsug.org/wp-content/uploads/2015/10/Montagna-Using-SAS-to-Manage-Biological-Species-Data-and-Calculate-Diversity-Indices.pdf
```

data species;
	set species;
	format sample $40.;
	sample=compress(treatment||locality||crop_cycle||bloc||plot||"_"||notation_date);
run;

data sp;
	set species;
	rename esp = species;
	rename cover = value;
	keep sample esp cover;
run;

PROC SORT DATA=sp;
	BY sample species;
RUN;
PROC TRANSPOSE DATA=sp OUT=tsp PREFIX=S;
	BY sample;
	ID species;
	VAR value;
RUN;
PROC TRANSPOSE DATA=tsp OUT=sp1;
	BY sample;
RUN;
DATA sp2;
	SET sp1;
	IF value=. THEN value=0;
RUN;

PROC SORT DATA=sp;
	BY sample species;
RUN;
PROC TRANSPOSE DATA=sp OUT=tsp2 PREFIX=S;
	BY sample;
	VAR value;
RUN;

DATA diversity;
	SET tsp2;
	ARRAY s s1-s100; /* Create temporary species array */
	DROP s1-s100 _name_;
	DO OVER s; /* Loop to convert zero to missing */
 		IF s=0 THEN s=.;
 	END;
	tn=sum(of s1-s100); /* tn = total ground cover of weed species */
	R=n(of s1-s100); /* R = Richness, total number of weed species EQ-1 */
	Hprime=0; /* Define all indices and set to zero */
	E1=0; 
	DO OVER s; /* Loop to calculate diversity indices */
 		If s=. THEN GOTO msp; /* Skip species with missing values */
 		Hprime = Hprime + (-(s/tn) * log(s/tn)); /* Shannon H’ index EQ-5 */
 		msp: ;
 	END;
	E1=Hprime/log(R); /* Eveness index, J’, EQ-8 */
	OUTPUT; /* Add new calculated values to dataset */
	RETURN; /* Start the next data step */
	FORMAT Hprime E1 8.2;
	FORMAT tn R 8.2;
	DROP maxn;
RUN;

proc sort data=species; by sample locality crop_cycle treatment bloc plot; run;
proc means data=species noprint;
	var age;
	by sample locality crop_cycle treatment bloc plot;
	output out=design n=;
run;

proc sort data = diversity; by sample; run;
proc sort data = design; by sample; run;

data div_analysis;
	merge diversity design;
	by sample;
	keep locality crop_cycle treatment bloc plot Hprime R E1;
run;

proc sort data=div_analysis;by locality crop_cycle treatment bloc plot;run;
proc means data=div_analysis noprint;
	var Hprime R E1;
	by locality crop_cycle treatment bloc plot;
	output out=div_analysis2 mean=;
run;

```
### Data analysis
#### Table 3: statistical analysis
```
%macro tem(note);
ods output Diffs = cmm_&note ;
proc mixed data=div_analysis2 covtest ;
	class locality crop_cycle bloc treatment;
	model &note=  treatment bloc(locality*crop_cycle) locality treatment*locality/outp=resid_&note s;
	repeated crop_cycle /subject=treatment*bloc*locality group=locality type=cs;
	lsmeans treatment/pdiff adjust=tukey ;
	lsmeans treatment*locality/slice=locality pdiff adjust=tukey;
run;
	data cmm_&note;
		set cmm_&note;
		where locality = _locality;
	run;
%mend;
%tem(R);
%tem(e1);
%tem(Hprime);
```
