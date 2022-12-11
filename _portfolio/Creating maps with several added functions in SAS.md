---
title: "Creating maps with several added functions in SAS"
excerpt: ""
collection: portfolio
---
# Introduction

Creating actual maps of countries and their states or municipialities can be a interesting way of showcasing data across a geospatial dimension. Almost everything that can be desired in this regard can be done in SAS with the ´g´-functions. A heatmap should be able to show the diversity across the regions, furthermore added functions can be applied to enhance the program and output that is desired. 

In the section below, a heatmap of Denmark and it's municipilaties are created in SAS. The program could be modified to showcase another country and/or regions in that country. Some interesting functions are added to the program, such that the output can be customized and enhanced:
- Automatically generated intervals for the heatmap (based on Nelders algorithm), such that the map shows diversity
- Automatically convert input datavalue to numerical if it is a character-value (if it can be converted, e.g. is "0.53")
- A format can be applied to the values if desired (possible to set intervals for the displayed value)
- Different color schemes for categorical and discrete variables
- Zoomed inset of regions that are small and/or lie far out from the mainland
- Output image as PDF or PNG
- Map function saved as a permanent macro using `mstored` , meaning that it can be easily called in other programs. 

The map functions ends up looking something like:

```sas
	%map(
		Dataset = 			/* Mandatory. Dataset. */
		,Var =				/* Mandatory. Variable to plot. */
		,output_path = 			/* Optional. Output path if PDF is desired. */
		,ID =				/* Optional. ID code, e.g. state name. Standard: ID */ 
		,discrete_or_cat =      	/* Optional. Discrete or categorical variable. Standard: Discrete */
		,nlevels =			/* Optional. Number of intervals. Standard: 5. Standard: 5 */
		,format = 			/* Optional. Display value format. Standard: commax10.2. */
		,miss_color			/* Optional. Missing data color. Standard: yellow */
				)
```

To which the finished product could look something like (using automatically generated intervals):

![[MapExample.png|600]]

# Construction
Only the essential parts, message me if interested in more :) 

Using the maps already integrated in SAS, the different parts of the map has to be allocated to different datasets to display them in different positions on the map.
This means based on the ID (municipial code in this example), create different datasets:

```sas
/* Mainland Danmark */
data dk_map;
	set mapsgfk.denmark;
	&ID. = substr(ID,4,3)*1;
	if &ID. in (400,411) then delete;
run;

/* Bornholm */
data dk_map_born;
	set mapsgfk.denmark;
	&ID. = substr(ID,4,3)*1;
	if &ID. not in (400) then delete;
run;

/* Christiansø */
data dk_map_chris_oe;
	set mapsgfk.denmark;
	&ID. = substr(ID,4,3)*1;
	if &ID. not in (411) then delete;
run;

/* København */
data dk_map_kbh;
	set mapsgfk.denmark;
	&ID. = substr(ID,4,3)*1;
	if &ID. not in (
						 101
						,147
						,151
						,153
						,155
						,157
						,159
						,161
						,163
						,165
						,167
						,173
						,175
						,183
						,185
						,187
						,190
								) then delete;
run;
```

Then import the data that we wish to display in the map. A conversion is added so that if the variable is a character, and can be converted to numerical, it is converted:

```sas
data &Dataset._map_temp;
	set &Dataset.;
	if &var. = . then delete;
	vartype = VTYPE(&ID.);
	if vartype = "C" then do;
		&ID._ = &ID.*1;
		drop &ID.;
		rename &ID._ = &ID.;
	end;
	if vartype = "N" then do;
		&ID._ = &ID.;
		drop &ID.;
		rename &ID._ = &ID.;
	end;
	format &ID._;
run;
```

Both the map data and the data that is to be displayed in the map are now imported. With the `proc gmap` procedure now display the maps. All the maps from the datasets we previously are created, such that they at the end can be imported into the same image. An example of the map creation could look like:

```sas
/* Genererer kort af Mainland Danmark */
goptions xpixels=1000 ypixels=1000;
proc gmap
	map=dk_map
    data=&Dataset._map_temp all;
	format &var. &format.;
  	id &ID.;
   	choro &var. /
		legend=legend1
		coutline = black
		cdefault = &miss_farve.
		levels = &nlevels.
		midpoints = OLD RANGE
		name = "DK"
;
	title ls=1.5 "Danmarkskort";
		title2 angle=-90 height=25pct ' '; /* Dummy titel der skubber kortet til siden */

	note 
		move=(5,95)pct
		justify = left 
		height=1 
		"Viser heatmap af variabel &var."
	j=l
		move=(5,93)pct
		justify = left 
		height=1
		"fordelt på kommuner";
run;
title;
title2;
footnote;
```

and

```sas
/* Genererer kort af Bornholm */
footnote1 h=15pct "Bornholm";

goptions xpixels=100 ypixels=100;
proc gmap map=dk_map_born data=testdata all;
format &var. &format.;
id &id.; 
choro &var. / 
		nolegend
		coutline = black
		cdefault = &miss_farve.
		levels = &nlevels.
		midpoints = OLD RANGE
		name='born'
		;
run;
footnote;
```

Repeat for all intended map parts.

These maps are named with a corresponding name, e.g. 'born' for bornholm. Using the `proc greplay` procedure to display all the maps on top of each other as:

```sas
/* Kører greplay */
proc greplay tc=tempcat nofs igout=work.gseg;
  tdef crmap des='crmap'
   1/llx = 0   		lly = 0
     ulx = 0   		uly = 100
     urx = 100  	ury = 100
     lrx = 100  	lry = 0

   2/llx = 76  		lly = 53 
     ulx = 76	  	uly = 60
     urx = 83 	 	ury = 60
     lrx = 83	  	lry = 53
	 color=black

   3/llx = 83  		lly = 53
     ulx = 83	  	uly = 60
     urx = 90	  	ury = 60
     lrx = 90	  	lry = 53
	 color=black

   4/llx = 60  		lly = 60 
     ulx = 60	  	uly = 90
     urx = 90	  	ury = 90
     lrx = 90	  	lry = 60
	 color=black

;
template = crmap;
treplay
 1:DK
 2:born
 3:chris_oe
 4:kbh
 ;
run;
```

This will insert the maps into the same picture. The sizes of the x and y coordinates should correspond to the sizes of the map image set by e.g. `goptions xpixels=100 ypixels=100;`

That's the main structure of the programme. Several additions can be made to add the functions previously mentioned in the introduction. 







