# Command-line tools to prepare Luna inputs

- terminal app & bash
- text editor
- package manager / instaling software
- network access (e.g. SSH) 
- EDF as input

- easiest - docker


## Key command tools

| Goal | Tool(s) |
|------|-------|
| Work with files and folders |
| Manipulate/reformat data |  `awk` |
| Variables | |
| Loops | |
| Advanced manipualtions | `R` | 

### Man


## Preparing a project

10 studies, EDFs
Annotations as Excel files, in different formats

Will go through the steps of making an analysis-ready Luna project

#### 


#### IDs


# points

XLS -> TXT
spaces in names


!!! info
    notes on best practice for IDs



library(readxl)
setwd("/data/purcell/projects/mesa/xls")
xls_files <- list.files(pattern = "\\.xls$")
    d <- read_excel(file)
    write.table(d1,
        file = paste("/data/purcell/projects/mesa/annot", id, ".annot", sep = ""),
        row.names = F, col.names = F, quote = F, sep = "\t"



4018176-20230906    # extra field
3011623-20230607    # small, no staging
3011984-20230705    # edittde cols
3010228-20230403    # norm
3010082-20230620    # norm

3010082-20230620_Event Grid.xls
3010228-20230403_Event Grid.xls
3011623-20230607_Event Grid_Less than Standard Quality.xls
4018176-20230906_Event Grid.xls
3011984-20230705_Event Grid.xls


eriscp localhost:/data/nsrr/working/mesa-exam7-edf/4018176-20230906*edf .
eriscp localhost:/data/nsrr/working/mesa-exam7-edf/3011623-20230607*edf .
eriscp localhost:/data/nsrr/working/mesa-exam7-edf/3011984-20230705*edf .
eriscp localhost:/data/nsrr/working/mesa-exam7-edf/3010228-20230403*edf .
eriscp localhost:/data/nsrr/working/mesa-exam7-edf/3010082-20230620*edf .


# 1) XLS to text

mkdir txt

library( readxl )
files <- list.files( "xls" , pattern = ".xls" , full.names = T )

for ( file in files ) {
d <- read_xls( file )
txt.file <- gsub( "xls" , "txt" , file )
write.table( d , file = txt.file , row.names=F , col.names=T , quote=F , sep="\t" ) 
}


> file = files[1]
> d <- read_xls( file )
> head(d)                                                                     
# A tibble: 6 × 6
  Event          Duration      `Start Time` `End Time` `Start Epoch` `End Epoch`
  <chr>          <chr>         <chr>        <chr>      <chr>         <chr>      
1 []             [s]           []           []         [#]           [#]        
2 Analysis Start 0             45097.95834… 45097.958… 0             0          
3 Artifact       937.97999999… 45097.95835… 45097.969… 0             31         
4 Supine         2140          45097.95839… 45097.983… 0             71         
5 Wake           30            45097.95868… 45097.959… 1             2          
6 Wake           30            45097.95902… 45097.959… 2             3          



