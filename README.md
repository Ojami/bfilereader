[![View bfilereader on File Exchange](https://www.mathworks.com/matlabcentral/images/matlab-file-exchange.svg)](https://se.mathworks.com/matlabcentral/fileexchange/96827-bfilereader)
# bfilereader :fire:
`bfilereader` (short for big file reader) is a MATLAB tool for reading and parsing big delimited files (can also be GNU zip `gz` compressed) in a fast and efficient manner. `bfilereader` takes advantages of a dependent Java class (`bFileReaderDep.class`) with multiple methods implemented for reading, mapping and filtering big delimited files.

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
Different uses of `bfilereader` have been covered in the following examples. The working file for these examples is publicly available [GWAS summary statistics on iron metabolism disorder](https://pheweb.org/UKB-SAIGE/download/275.1) which can be freely download in GZIP format (``phenocode-275.1.tsv.gz``). 
```matlab
info = dir('phenocode-275.1.tsv.gz');
fprintf('file size: %.2f mb\n', info.bytes/1e6)
file size: 641.33 mb
```
### Example 1: a quick glance at file content
To see what sort of data this file contains, we can simply call the function with `summary` option set to `only` to fetch _only_ first few lines. It's also useful to see the number of rows by setting `verbose` flag to `on`.
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
This file has ~28,300,000 rows and 13 columns, which *may* not fit into the memory. From the first row, we easily notice that the file has one header row (variable names). We can use this summary table to extract additional data.

### Example 2: pattern matching
Function can apply pattern matching to extract desired information. For instance, to extract variants on either PNPLA3 or GPAM genes (column `nearest_genes`):
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
We can also use regular expression (regex) and further add another filter. This time, we want to find 1)all variants on PNPLA family genes and 2)missense variants on other genes. We also would like to only extract frist 10 columns (by using `extractCol` option). 
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
In above example, we fetched all missense variants except for PNPLA family, for which we extracted all variants regardless of their consequences. But, what if we were only interested in finding missense variants on PNPLA family genes? To do so, we need to set ``multiCol`` flag to `true` to tell `bfilereader` to apply each pattern to its corresponding column (i.e. pattern 1 to column 1, pattern 2 to column 2, ...):
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'pattern', patterns, 'patternCol', cols,...
     'extractCol', 1:10, 'multiCol', true);
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
In addition to pattern matching for strings, data of numeric type can be filtered as well. For instance, if we would like to only see how many variants pass genome-wide significance threshold (`5e-8`), we can set ``filter`` and `filterCol` options:
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'extractCol', 1:10,...
     'filter', 5e-8, 'filterCol', "pval", 'operator', '<='); % any pval <= 5e-8
Elapse time: 26.312 sec

size(out)
  10598          10

fprintf('pval range: %.3g - %.3g\n', min(out.pval), max(out.pval))
pval range: 0 - 5e-08
```
Similar to pattern matching, we can filter for other columns. However, we don't need to set `multiCol` option in this case since every filtering value can only be applied to one column. This time we want to find variants passing genome-wide significance threshold and having a negative effect size (beta):
```
out = bfilereader('phenocode-275.1.tsv.gz', 'header', true, 'extractCol', 1:10, ...
     'filter', [5e-8, 0], 'filterCol', ["pval", "beta"], 'operator', '<='); % any pval <= 5e-8
Elapse time: 27.426 sec

size(out)
  7806          10

fprintf('pval range: %.3g - %.3g\n', min(out.pval), max(out.pval))
fprintf('beta range: %.3g to %.3g\n', min(out.beta), max(out.beta))

pval range: 1.5e-197 - 5e-08
beta range: -2.9 to -0.3
```

### Example 4: Pattern matching with numeric filtering
We can also use a mixture of examples 2 and 3 by including both filtering and pattern options. Suppose we want to get missense variants on [MHC class I genes](https://en.wikipedia.org/wiki/MHC_class_I) passing 5e-8 threshold and having a negative beta:
```
patterns = ["HLA-\w{1}$", "missense"]; 
cols = ["nearest_genes", "consequence"];
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

## Additional notes
``bfilereader`` can also parse the input file in parallel; however, whether using this option positively or negatively influences the performance depends on several factors (size of file in hand, the available memory, memory overhead and more). For a good discussion, [see here](https://levelup.gitconnected.com/be-careful-with-java-parallel-streams-3ed0fd70c3d0). To show how it may affect the file processing, we consider 3 scenarios using different delimited files and we compare sequential and parallel ``bfilereader`` with MATLAB tall datastore. Under all scenarios, we will use only uncompressed files.

### Scenario 1: ~490 MB 
We begin with a [similar but smaller GWAS summary statistics file](https://pheweb.org/MGI-freeze1/pheno/286.81). 
```
file = "phenocode-286.81.tsv";

fprintf('file size: %.2f mb\n', dir(file).bytes/1e6)
ile size: 490.55 mb

patt = ["LDLR$", "missense"];
col = ["nearest_genes", "consequence"];
out = bfilereader(file, 'header', true, 'pattern', patt, 'patternCol', col, 'multiCol', true); % sequential

out = bfilereader(file, 'header', true, 'pattern', patt, 'patternCol', col, 'multiCol', true, 'parallel', true); % parallel

% check with MATLAB tall datastore
parpool('local', 8); % max available cores
ds = tabularTextDatastore(file, 'FileExtensions', '.tsv', 'TextType', 'string');
ds.SelectedFormats{1} = '%q'; % for chromosome "X"
ds = tall(ds);
idx = endsWith(ds.(col(1)), "LDLR") & contains(ds.(col(2)), patt(2)); % regexp and contains cannot be applied to tall arrays
out2 = gather(ds(idx, :));

% benchmark: mean elapsed time over 3 repetitions
tbench.sequential = [7.3670 7.2930 7.3010];
tbench.parallel = [3.7540 3.7330 3.8020];
tbench.tall = [5.6000 5.4000 5.2000];
disp(table(structfun(@mean, tbench), 'VariableNames', {'mean elapsed time'}, 'RowNames', fieldnames(tbench)))

                  mean elapsed time
                  _________________

    sequential         7.3203      
    parallel            3.763      
    tall                  5.4


```
### Scenario 2: ~2.3 GB
Next, we use the same delimited but uncompressed file (`phenocode-275.1.tsv`) we used in examples above. This file is relatively bigger (~4.6 times) than the file we used in scenario 1.
```
file = "phenocode-275.1.tsv";

fprintf('file size: %.2f GB\n', dir(file).bytes/1e9)
file size: 2.43 GB

patterns = ["HLA-", "missense"];
cols = ["nearest_genes", "consequence"];
out = bfilereader(file, 'header', true, 'extractCol', 1:10, 'filter', [5e-8, 0], 'filterCol',["pval", "beta"], 'operator', "<=",...
     'pattern', patterns, 'patternCol', cols, 'multiCol', true);

out = bfilereader(file, 'header', true, 'extractCol', 1:10, 'filter', [5e-8, 0], 'filterCol',["pval", "beta"], 'operator', "<=",...
     'pattern', patterns, 'patternCol', cols, 'multiCol', true, 'parallel', true);

ds = tabularTextDatastore(file, 'FileExtensions', '.tsv', 'TextType', 'string');
ds.SelectedFormats{1} = '%q';
tt = tall(ds);
idx = startsWith(tt.(cols(1)), patterns(1)) & startsWith(tt.(cols(2)), patterns(2)) & tt.pval <= 5e-8 & tt.beta <= 0;
out2 = gather(ds(idx, :));

% benchmark: mean elapsed time over 3 repetitions
tbench.sequential = [30.593, 30.447, 30.371];
tbench.parallel = [19.740 19.907 20.116];
tbench.tall = [23.361, 25.508, 24.617];
disp(table(structfun(@mean, tbench), 'VariableNames', {'mean elapsed time'}, 'RowNames', fieldnames(tbench)))
                  mean elapsed time
                  _________________

    sequential          30.47      
    parallel           19.921      
    tall               24.495  
```
### Scenario 3: ~18 GB
Lastly, we use a much bigger file (I used chromosome 1 from [dbNSFP project](https://sites.google.com/site/jpopgen/dbNSFP) version 4.1). In this case, ``parallel`` would throw [out of memory error](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html), so we only benchmark with sequential ``bfilereader`` and tall datastore. 
```
file = "dbNSFP4.1a_variant.chr1";

fprintf('file size: %.2f GB\n', dir(file).bytes/1e9)
file size: 18.44 GB

filterCol = "CADD_phred";
patternCol = "genename";
patt = "^MARC1$";
filter = 15;

out = bfilereader(file, 'header', true, 'pattern', patt, 'patternCol', patternCol,...
     'filter', filter, 'filterCol', filterCol, 'operator', '>=');


parpool('local', 8); % max available cores
ds = tabularTextDatastore(file, 'FileExtensions', '.chr1', 'TextType', 'string', 'TreatAsMissing', {'.', '-'});
ds.SelectedFormats = repmat({'%q'}, 1, numel(ds.SelectedFormats)); % to avoid conversion to double error
ds.SelectedFormats(ismember(ds.SelectedVariableNames, filterCol)) = {'%f'};
tt = tall(ds);
idx = ismember(tt.(patternCol), "MARC1") & tt.(filterCol) >= filter;
out2 = gather(ds(idx, :));

% benchmark: mean elapsed time over 3 repetitions
tbench.sequential = [113.885, 113.662, 113.943];
tbench.tall = [389.2, 392.66 405.336];
disp(table(structfun(@mean, tbench), 'VariableNames', {'mean elapsed time'}, 'RowNames', fieldnames(tbench)))
mean elapsed time
                  _________________

    sequential         113.83      
    tall               395.73   
```

In scenarios 1 and 2, parallel ``bfilereader`` performed better than both MATLAB tall datastore and sequential ``bfilereader``. However, when file does not fit into the memory like the case in scenario 3, memory overhead can be a serious issue. Under such circumstances, parallel computing can negatively affect the performance. Therefore, don't apply ``parallel`` flag just because it seems cool! proper benchmarking can show if your task really benefits from parallel computing or not.

