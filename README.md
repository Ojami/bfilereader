# bfilereader :fire:
`bfilereader` (short for big file reader) is a MATLAB function for reading and parsing big delimited files (can also be GNU zip `gz` compressed) in a fast and efficient manner. `bfilereader` takes advantages of a dependent Java class (`bFileReaderDep.class`) with multiple methods implemented for reading, mapping and filtering big delimited files.

## Requirements
There are few requirements to be met prior to use `bfilereader`:
- MATLAB version must be **R2019b or newer**.
- ``bFileReaderDep`` was writtent and compiled in [Java 8](https://www.oracle.com/java/technologies/java8.html). For more information see [MATLAB documents on Java Environment](mathworks.com/help/compiler_sdk/java/configure-your-java-environment.html).
- To avoid [Java out of memory issues](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html) when working with large files, configure and increase your [MATLAB Java Heap Memory Preferences](mathworks.com/help/matlab/matlab_external/java-heap-memory-preferences.html).


## Installation
Add the both `bfilereader.m` and `bFileReaderDep.class` to your MATLAB path.
```
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
```
info = dir('phenocode-275.1.tsv.gz');
fprintf('file size: %.2f mb\n', info.bytes/1e6)
file size: 641.33 mb
```
### A quick glance at file content
To see what sort of data is stored in this file, we can simply only call the function with `summary` option set to `only` to ask _only_ for few first lines of the file.
```
out = bfilereader('phenocode-275.1.tsv.gz', 'summary', 'only');
  Var1_1     Var1_2     Var1_3    Var1_4       Var1_5            Var1_6                 Var1_7             Var1_8    Var1_9     Var1_10      Var1_11      Var1_12      Var1_13 
    _______    _______    ______    ______    _____________    _______________    _______________________    ______    _______    ________    _________    __________    ________

    "chrom"    "pos"      "ref"     "alt"     "rsids"          "nearest_genes"    "consequence"              "pval"    "beta"     "sebeta"    "af"         "ac"          "tstat" 
    "1"        "16071"    "G"       "A"       "rs541172944"    "OR4F5"            "intron_variant"           "0.71"    "-2.8"     "7.6"       "5e-05"      "40.9"        "-0.048"
    "1"        "16280"    "T"       "C"       "rs866639523"    "OR4F5"            "intron_variant"           "0.56"    "-2.3"     "3.9"       "0.00015"    "125.6"       "-0.15" 
    "1"        "49298"    "T"       "C"       "rs10399793"     "OR4F5"            "upstream_gene_variant"    "0.66"    "0.042"    "0.097"     "0.62"       "508337.5"    "-20.0" 
    "1"        "54353"    "C"       "A"       "rs140052487"    "OR4F5"            "intron_variant"           "0.8"     "0.81"     "3.3"       "0.00036"    "289.1"       "0.076" 
    "1"        "54564"    "G"       "T"       "rs558796213"    "OR4F5"            "intron_variant"           "0.98"    "0.091"    "3.1"       "0.00015"    "121.2"       "0.0095"
```

## Benchmarking
