# Configuration Report
The format of a configuration output file is, by design, identical to a configuration input file. The details of that format can be found in the corresponding section of the [previous](https://ritchielab.github.io/biofilter-manual/04_InputOutputFormats/InputOutput.html#configuration-files-not-required) chapter.

Note however that the `INCLUDE` instruction is not relevant for configuration output files because the structure of inclusions is not preserved internally. This means that even if the configuration file(s) provided to Biofilter include other configuration files, the report generated by the `--report-configuration` option will not contain any `INCLUDE` instructions. Instead, all options from all included files will be merged into a single reported configuration.

# Gene and Group Name Statistics Reports
These reports list all of the types of identifiers available for genes or groups with `--report-gene-name-stats`/`--report-group-name-stats` option, respectively, along with some statistics about their overall uniqueness. 

Examples (TODO need to update numbers?):
```
#type           names       unique      ambiguous
symbol          254375      251386      2989
entrez_gid      213675      213675      0
refseq_gid      281129      281129      0
refseq_pid      199675      199675      0
ensembl_gid     95531       95416       115
ensembl_pid     47036       47036       0
hgnc_id         43649       43649       0
mim_id          18303       18303       0
mirbase_id      1916        1916        0
unigene_gid     29208       28066       1142
uniprot_pid     120251      118931      1320
pharmgkb_gid    24526       24526       0
```

# LD Profiles Report
This report lists the default LD profiles with canonical gene boundaries available in the [knowledge database](https://ritchielab.github.io/biofilter-manual/loki/loki.html) by the `--report-ld-profiles` option.

# Invalid Input Report
If the `--report-invalid-input` option has been enabled, then any user input data which cannot be parsed or understood by Biofilter will appear in one of these report files. A separate file is generated for each type of input (SNP, position, region, etc.), and for each invalid input line, that entire line will be copied to the corresponding report file preceded by a comment line describing the error. For example, the SNP input file:
```
#snp
rs12
rs34
chr5:678
rs90
```
will yield the invalid SNP report file:
```
# invalid literal for int() with base 10: 'chr5:678' at index 3
chr5:678
```
One of the inputs was not understood as a valid RS number, but the other three were parsed successfully and added to the input dataset.

# Analysis Outputs
*Annotation*, *filtering*, and *modeling* analyses always return one or more tab-separated columns, but the number and contents of those columns can vary. Each analysis mode allows the user to exactly specify the desired output columns.

In the simplest case, the user can request one of the six data types which Biofilter also takes as input: *SNP*, *position*, *region*, *gene*, *group* or *source*. The output will then contain one or more columns describing the specified data type, in exactly the same format as Biofilter requires for input of the same type. For example, SNP output produces a single column of RS numbers, position output produces three columns (chromosome, label, position), and so on. More than one basic type can also be output together (and is required for annotation and modeling analyses), in which case the columns corresponding to any additional types are simply appended in order to the final output. For example, the analysis options on the left will produce the output columns on the right:

|**Analysis options**|**Output columns**|
|---|---|
|`--annotate gene region`|gene&emsp;chr&emsp;region&emsp;start&emsp;stop|
|`--filter position`|chr&emsp;position&emsp;pos|
|`--filter gene snp`|gene&emsp;snp|
|`--model gene`|gene1&emsp;gene2&emsp;score|

Note that there are two different places Biofilter could draw from when outputting any given type of data: one of the user input datasets, or the prior knowledge database. If an output type is requested which was not provided as input then the choice is clear, and Biofilter will produce the requested output based on the data contained in the knowledge database. For any data type which was provided as input, however, Biofilter will pull any corresponding output columns from the input data rather than the knowledge database.

This means, for example, that if regions are supplied as input and both “`gene`” and “`region`” are requested as output, then the result may not be as expected. The output will list both genes and regions, and the genes in the first column will indeed be the ones whose genomic region matched one of the provided input regions. However, the region shown next to each gene will not be that gene’s region, as one might hope; it will instead be the user-provided input region which matched the gene’s region.

Biofilter provides additional output options to deal with situations such as these. The six basic types will suffice for most use cases, but they are actually only shorthand for their respective sets of individual output columns. For more particular use cases there are a few additional shorthand types (such as “`generegion`”), and any single output column may also be requested individually. This includes each
separate column from any of the six data type outputs (such as “`region_chr`” which is the first of four columns included in the “`region`” output type), as well as some columns which are not included in any of the shorthand sets.

Biofilter currently supports [the following outputs](https://ritchielab.github.io/biofilter-manual/05_BiofilterArguments/BiofilterArguments.html#types-for-analysis-option-arguments).

Inspection of Biofilter’s source code may reveal additional supported columns. They are not documented here because they are only used for internal or debugging purposes and may change or disappear in a future release; use them at your own risk.