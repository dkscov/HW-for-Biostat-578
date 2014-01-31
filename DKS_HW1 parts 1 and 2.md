David Scoville - HW 1
========================================================

Step 1: Connect to the GEO database


```r
library(GEOmetadb)
geo_con<-dbConnect(SQLite(),'GEOmetadb.sqlite')
```


Step 2: Check what tables are in GEO


```r
dbListTables(geo_con)
```

```
 [1] "gds"               "gds_subset"        "geoConvert"       
 [4] "geodb_column_desc" "gpl"               "gse"              
 [7] "gse_gpl"           "gse_gsm"           "gsm"              
[10] "metaInfo"          "sMatrix"          
```



```r
library(GEOquery)

dbGetQuery(geo_con,"SELECT gse.title, gse.gse, gpl.gpl, gpl.manufacturer,gpl.description, gse.contact FROM (gse JOIN gse_gpl ON gse.gse = gse_gpl.gse) j JOIN gpl ON j.gpl= gpl.gpl WHERE gse.title like'%HCV%' AND gse.contact like'%YALE%' AND gpl.manufacturer like'%Illumina%'")
```

```
                                                           gse.title
1 The blood transcriptional signature of chronic HCV [Illumina data]
2                 The blood transcriptional signature of chronic HCV
   gse.gse  gpl.gpl gpl.manufacturer
1 GSE40223 GPL10558    Illumina Inc.
2 GSE40224 GPL10558    Illumina Inc.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      gpl.description
1 The HumanHT-12 v4 Expression BeadChip provides high throughput processing of 12 samples per BeadChip without the need for expensive, specialized automation. The BeadChip is designed to support flexible usage across a wide-spectrum of experiments.;\t;\tThe updated content on the HumanHT-12 v4 Expression BeadChips provides more biologically meaningful results through genome-wide transcriptional coverage of well-characterized genes, gene candidates, and splice variants.;\t;\tEach array on the HumanHT-12 v4 Expression BeadChip targets more than 31,000 annotated genes with more than 47,000 probes derived from the National Center for Biotechnology Information Reference Sequence (NCBI) RefSeq Release 38 (November 7, 2009) and other sources.;\t;\tPlease use the GEO Data Submission Report Plug-in v1.0 for Gene Expression which may be downloaded from https://icom.illumina.com/icom/software.ilmn?id=234 to format the normalized and raw data.  These should be submitted as part of a GEOarchive.  Instructions for assembling a GEOarchive may be found at http://www.ncbi.nlm.nih.gov/projects/geo/info/spreadsheet.html;\t;\tOctober 11, 2012: annotation table updated with HumanHT-12_V4_0_R2_15002873_B.txt
2 The HumanHT-12 v4 Expression BeadChip provides high throughput processing of 12 samples per BeadChip without the need for expensive, specialized automation. The BeadChip is designed to support flexible usage across a wide-spectrum of experiments.;\t;\tThe updated content on the HumanHT-12 v4 Expression BeadChips provides more biologically meaningful results through genome-wide transcriptional coverage of well-characterized genes, gene candidates, and splice variants.;\t;\tEach array on the HumanHT-12 v4 Expression BeadChip targets more than 31,000 annotated genes with more than 47,000 probes derived from the National Center for Biotechnology Information Reference Sequence (NCBI) RefSeq Release 38 (November 7, 2009) and other sources.;\t;\tPlease use the GEO Data Submission Report Plug-in v1.0 for Gene Expression which may be downloaded from https://icom.illumina.com/icom/software.ilmn?id=234 to format the normalized and raw data.  These should be submitted as part of a GEOarchive.  Instructions for assembling a GEOarchive may be found at http://www.ncbi.nlm.nih.gov/projects/geo/info/spreadsheet.html;\t;\tOctober 11, 2012: annotation table updated with HumanHT-12_V4_0_R2_15002873_B.txt
                                                                                                                                                                                                                                                     gse.contact
1 Name: Christopher Bolen;\tEmail: christopher.bolen@yale.edu;\tLaboratory: Steven Kleinstein;\tDepartment: Pathology;\tInstitute: Yale University;\tAddress: 300 George Street, Suite 505;\tCity: New Haven;\tState: CT;\tZip/postal_code: 06511;\tCountry: USA
2 Name: Christopher Bolen;\tEmail: christopher.bolen@yale.edu;\tLaboratory: Steven Kleinstein;\tDepartment: Pathology;\tInstitute: Yale University;\tAddress: 300 George Street, Suite 505;\tCity: New Haven;\tState: CT;\tZip/postal_code: 06511;\tCountry: USA
```


Using data.table

Step 1: make data tables

```r
library(data.table)
gse_gpl.dt<-as.data.table(dbGetQuery(geo_con, "SELECT * FROM gse_gpl"))
gse.dt<-as.data.table(dbGetQuery(geo_con, "SELECT * FROM gse"))
gpl.dt<-as.data.table(dbGetQuery(geo_con, "SELECT * FROM gpl"))
```

Step 2: Setting keys and merging the tables


```r
setkey(gse.dt, gse)
setkey(gse_gpl.dt, gse)
setkey(gpl.dt,gpl)
merge1<-(merge(gse.dt,gse_gpl.dt))
setkey(merge1,gpl)
merge2<-merge(merge1,gpl.dt)
```

Step 3: Querying the data.table for HCV data from Investigator at Yale who used Illumina

```r
merge2[title.x %like% "HCV" & contact.x %like% "Yale"& manufacturer %like% "Illumina",list(title.x,gse,gpl,manufacturer,description)]
```

```
                                                              title.x
1: The blood transcriptional signature of chronic HCV [Illumina data]
2:                 The blood transcriptional signature of chronic HCV
        gse      gpl  manufacturer
1: GSE40223 GPL10558 Illumina Inc.
2: GSE40224 GPL10558 Illumina Inc.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           description
1: The HumanHT-12 v4 Expression BeadChip provides high throughput processing of 12 samples per BeadChip without the need for expensive, specialized automation. The BeadChip is designed to support flexible usage across a wide-spectrum of experiments.;\t;\tThe updated content on the HumanHT-12 v4 Expression BeadChips provides more biologically meaningful results through genome-wide transcriptional coverage of well-characterized genes, gene candidates, and splice variants.;\t;\tEach array on the HumanHT-12 v4 Expression BeadChip targets more than 31,000 annotated genes with more than 47,000 probes derived from the National Center for Biotechnology Information Reference Sequence (NCBI) RefSeq Release 38 (November 7, 2009) and other sources.;\t;\tPlease use the GEO Data Submission Report Plug-in v1.0 for Gene Expression which may be downloaded from https://icom.illumina.com/icom/software.ilmn?id=234 to format the normalized and raw data.  These should be submitted as part of a GEOarchive.  Instructions for assembling a GEOarchive may be found at http://www.ncbi.nlm.nih.gov/projects/geo/info/spreadsheet.html;\t;\tOctober 11, 2012: annotation table updated with HumanHT-12_V4_0_R2_15002873_B.txt
2: The HumanHT-12 v4 Expression BeadChip provides high throughput processing of 12 samples per BeadChip without the need for expensive, specialized automation. The BeadChip is designed to support flexible usage across a wide-spectrum of experiments.;\t;\tThe updated content on the HumanHT-12 v4 Expression BeadChips provides more biologically meaningful results through genome-wide transcriptional coverage of well-characterized genes, gene candidates, and splice variants.;\t;\tEach array on the HumanHT-12 v4 Expression BeadChip targets more than 31,000 annotated genes with more than 47,000 probes derived from the National Center for Biotechnology Information Reference Sequence (NCBI) RefSeq Release 38 (November 7, 2009) and other sources.;\t;\tPlease use the GEO Data Submission Report Plug-in v1.0 for Gene Expression which may be downloaded from https://icom.illumina.com/icom/software.ilmn?id=234 to format the normalized and raw data.  These should be submitted as part of a GEOarchive.  Instructions for assembling a GEOarchive may be found at http://www.ncbi.nlm.nih.gov/projects/geo/info/spreadsheet.html;\t;\tOctober 11, 2012: annotation table updated with HumanHT-12_V4_0_R2_15002873_B.txt
```



