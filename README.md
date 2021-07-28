# bfilereader :fire:
`bfilereader` (short for big file reader) is a MATLAB function for reading and parsing big delimited files (can also be GNU zip `gz` compressed) in a fast and efficient manner. `bfilereader` takes advantages of a dependent Java class (`bFileReaderDep.class`) with multiple methods implemented for reading, mapping and filtering big delimited files.

## Requirements
There are few requirements to be met prior to use `bfilereader`:
- MATLAB version must be **R2019b or newer**.
- ``bFileReaderDep`` was writtent and compiled in [Java 8](https://www.oracle.com/java/technologies/java8.html). For more information see [MATLAB documents on Java Environment](mathworks.com/help/compiler_sdk/java/configure-your-java-environment.html).
- To avoid [Java out of memory issues](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html) when working with large files, configure and increase your [MATLAB Java Heap Memory Preferences](mathworks.com/help/matlab/matlab_external/java-heap-memory-preferences.html).


## Installation
Add the both `bfilereader.m` and `bFileReaderDep.class` to your MATLAB path.
```matlab
addpath(pwd);
savepath;
```

## Overview
`bfilereader` can be used for different purposes:
- Reading the whole content or multiple columns of a delimited file (similar behavior but limited functionality to MATLAB [`readtable`](mathworks.com/help/matlab/ref/readtable.html)).
- Text pattern matching (with regular expression).
- Filtering data of numeric type based on different criteria.

## Examples
Different use of `bfilereader` have been covered in the following examples. The working file for these examples is publicly available [GWAS summary statistics on iron metabolism disorder](https://pheweb.org/UKB-SAIGE/download/275.1) which can be freely download in GZIP format (``phenocode-275.1.tsv.gz``). 
```matlab
info = dir('phenocode-275.1.tsv.gz');
fprintf('file size: %.2f mb\n', info.bytes/1e6)
file size: 641.33 mb
```
### Example 1: a quick glance at file content
To see what sort of data is stored in this file, we can simply only call the function with `summary` option set to `only` to ask _only_ for few first lines of the file. It's also useful to see the number of rows by setting `verbose` flag to `on`.
```
out = bfilereader('phenocode-275.1.tsv.gz', 'summary', 'only', 'verbose', 'on');
Elapsed time is 11.828086 seconds.
file has 28336915 lines and 13 columns
file first 6 rows:
     Var1       Var2      Var3     Var4         Var5              Var6                   Var7               Var8      Var9       Var10        Var11        Var12        Var13  
    _______    _______    _____    _____    _____________    _______________    _______________________    ______    _______    ________    _________    __________    ________

    "chrom"    "pos"      "ref"    "alt"    "rsids"          "nearest_genes"    "consequence"              "pval"    "beta"     "sebeta"    "af"         "ac"          "tstat" 
    "1"        "16071"    "G"      "A"      "rs541172944"    "OR4F5"            "intron_variant"           "0.71"    "-2.8"     "7.6"       "5e-05"      "40.9"        "-0.048"
    "1"        "16280"    "T"      "C"      "rs866639523"    "OR4F5"            "intron_variant"           "0.56"    "-2.3"     "3.9"       "0.00015"    "125.6"       "-0.15" 
    "1"        "49298"    "T"      "C"      "rs10399793"     "OR4F5"            "upstream_gene_variant"    "0.66"    "0.042"    "0.097"     "0.62"       "508337.5"    "-20.0" 
    "1"        "54353"    "C"      "A"      "rs140052487"    "OR4F5"            "intron_variant"           "0.8"     "0.81"     "3.3"       "0.00036"    "289.1"       "0.076" 
    "1"        "54564"    "G"      "T"      "rs558796213"    "OR4F5"            "intron_variant"           "0.98"    "0.091"    "3.1"       "0.00015"    "121.2"       "0.0095"
```
This file has around ~28,300,000 rows with 13 columns, which *may* not fit into the memory. From the first row, we can easily understand the file has one header row (variable names), which can be used for extracting required data from the file. We can use this summary to extract more data.

### Example 2: pattern matching
We can apply pattern matching to this table to extract desired information. For instance, to extract variants on either PNPLA3 or GPAM genes (column `nearest_genes`):
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'pattern', ["PNPLA3", "GPAM"], 'patternCol', "nearest_genes");
Elapse time: 28.334 sec

size(out)
  ans =
        7776          13

unique(out.nearest_genes)
ans = 
  3×1 string array
    "GPAM"
    "PNPLA3"
    "PNPLA3,SAMM50"
```
We can also use regular expression (regex) and further add another filter. So, this time we want to find all missense variants and all variants on genes from PNPLA family. We also would like to only extract frist 10 columns (by using `extractCol` option). 
```
patterns = ["PNPLA*", "missense"];
cols = ["nearest_genes", "consequence"]; 
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'pattern', patterns, 'patternCol', cols, 'extractCol', 1:10);
Elapse time: 31.639 sec

size(out)
      171838          10
      
head(out, 4)
    chrom       pos        ref    alt        rsids        nearest_genes       consequence        pval     beta     sebeta
    _____    __________    ___    ___    _____________    _____________    __________________    _____    _____    ______

      1      1.3903e+05    "G"    "A"    "rs751110858"    "AL627309.1"     "missense_variant"     0.79     -1.4      5.4 
      1      1.3906e+05    "G"    "A"    "rs568513188"    "AL627309.1"     "missense_variant"    0.039      4.6      2.2 
      1      7.3854e+05    "T"    "C"    "rs147999235"    "OR4F16"         "missense_variant"     0.95    0.045     0.78 
      1      8.6135e+05    "C"    "T"    "rs200686669"    "SAMD11"         "missense_variant"      0.7    -0.81      2.1 
      
% check variants on PNPLA2 gene
pnpla2 = out(out.nearest_genes == "PNPLA2", :);
head(pnpla2, 4)
    chrom       pos        ref    alt        rsids        nearest_genes          consequence          pval     beta     sebeta
    _____    __________    ___    ___    _____________    _____________    _______________________    ____    ______    ______

     11      8.1591e+05    "C"    "T"    "rs191403268"      "PNPLA2"       "upstream_gene_variant"    0.94    -0.054     0.78 
     11      8.1602e+05    "A"    "G"    "rs11246321"       "PNPLA2"       "upstream_gene_variant"    0.33       0.2      0.2 
     11      8.1615e+05    "G"    "A"    "rs546654103"      "PNPLA2"       "upstream_gene_variant"    0.64       0.4     0.85 
     11      8.1619e+05    "C"    "T"    "rs755386980"      "PNPLA2"       "upstream_gene_variant"    0.78     -0.68      2.4 

```
In above example, we fetched all missense variants except for PNPLA family, for which we extracted all variantes regardless of their consequences. We can narrow down our pattern matching and only extract missense variants on PNPLA family. To do so, we need to set ``multiCol`` flag to `true` to technically tell `bfilereader` to match each pattern to each respective column:
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'pattern', patterns, 'patternCol', cols, 'extractCol', 1:10, 'multiCol', true);
Elapse time: 40.223 sec

size(out)
  107    10

head(out, 4)
    chrom       pos        ref    alt        rsids        nearest_genes       consequence        pval     beta     sebeta
    _____    __________    ___    ___    _____________    _____________    __________________    ____    ______    ______

      6      3.6259e+07    "C"    "T"    "rs140585347"      "PNPLA1"       "missense_variant"     0.7     -0.95      2.5 
      6      3.6262e+07    "G"    "A"    "rs74946910"       "PNPLA1"       "missense_variant"    0.82      -1.5      6.6 
      6      3.6263e+07    "G"    "A"    "rs45524833"       "PNPLA1"       "missense_variant"    0.95    -0.029     0.49 
      6       3.627e+07    "A"    "G"    "rs371888522"      "PNPLA1"       "missense_variant"    0.61      -1.2      2.3 

unique(out.nearest_genes)
    "AC008878.2,PNPLA6"
    "PNPLA1"
    "PNPLA2"
    "PNPLA3"
    "PNPLA3,SAMM50"
    "PNPLA5"
    "PNPLA6"
    "PNPLA7"
    "PNPLA8"
    "PNPLA8,THAP5"
```

### Example 3: Numeric filtering 
In addition to pattern matching for strings, data of numeric type can be filtered as well. For instance, we would like to only see how many variants pass genome-wide significance threshold (`5e-8`). 
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'extractCol', 1:10, 'filter', 5e-8, 'filterCol', "pval", 'operator', '<='); % any pval <= 5e-8
Elapse time: 26.312 sec

size(out)
  10598          10

fprintf('pval range: %.3g - %.3g\n', min(out.pval), max(out.pval))
pval range: 0 - 5e-08
```
Similar to pattern matching, we can filter for other columns. However, we don't need to set `multiCol` option in this case since every filtering value can only be applied to one column. This time we want to find variants passing genome-wide significance threshold but also have a negative effect size (beta):
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'extractCol', 1:10, 'filter', [5e-8, 0], 'filterCol', ["pval", "beta"], 'operator', '<='); % any pval <= 5e-8
Elapse time: 27.426 sec

size(out)
  7806          10

fprintf('pval range: %.3g - %.3g\n', min(out.pval), max(out.pval))
fprintf('beta range: %.3g to %.3g\n', min(out.beta), max(out.beta))

pval range: 1.5e-197 - 5e-08
beta range: -2.9 to -0.3
```

### Example 4: Pattern matching with numeric filtering
We could also use a mixture of examples 2 and 3 by including both filtering and pattern options. Suppose we want to get missense variants on [MHC class I genes](https://en.wikipedia.org/wiki/MHC_class_I) passing 5e-8 threshold and having a negative beta:
```
patterns = ["HLA-\w{1}$", "missense"]; 
cols = ["nearest_genes", "consequence"]'
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'extractCol', 1:10, 'filter', [5e-8, 0], 'filterCol',...
  ["pval", "beta"], 'operator', "<=", 'pattern', patterns, 'patternCol', cols, 'multiCol', true);
Elapse time: 41.277 sec

disp(out)
    chrom       pos        ref    alt       rsids       nearest_genes       consequence         pval      beta     sebeta
    _____    __________    ___    ___    ___________    _____________    __________________    _______    _____    ______

      6      2.9692e+07    "C"    "G"    "rs2072895"       "HLA-F"       "missense_variant"    5.5e-09    -0.32    0.056 
      6      2.9693e+07    "C"    "T"    "rs1736924"       "HLA-F"       "missense_variant"    4.3e-52     -1.2     0.08 
      6       2.991e+07    "C"    "G"    "rs1143146"       "HLA-A"       "missense_variant"    6.5e-18    -0.48    0.056 
      6      2.9911e+07    "T"    "G"    "rs1059542"       "HLA-A"       "missense_variant"    1.2e-66     -1.5    0.088 
      6      3.0458e+07    "G"    "A"    "rs1264457"       "HLA-E"       "missense_variant"    4.4e-08    -0.31    0.056 
```

## Benchmarking
