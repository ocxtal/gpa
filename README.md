# Proposal of Graphical-Pairwise-Alignment (GPA) format

* 2016/6/17 first update, next and previous alignment name were moved to mandatory fields
* 2016/6/11 proposal of a GPA format, Hajime Suzuki


## Overview

Graphical pairwise alignment (GPA) format is a text-based line-separated structure to represent a set of pairwise local alignments between two graphically-represented sequences. Although the [SAM and BAM format](https://github.com/samtools/hts-specs) are widely used as de-facto standard output formats of the sequence alignment tools, the SAM and BAM format has a defect in lacking a path information on the read side, making it unable to represent graphically-represented query sequences without external files or complecated optional fields. The GPA format is designed to remove the defect and complexity of the SAM format, following the convention of the [GFA](https://github.com/GFA-spec/GFA-spec) format, representing each record in a line and each field in a record delimited with tabs (`'\t'`). The format is originally used as a default output format of the [comb aligner](https://github.com/ocxtal/comb) and [libaw](https://github.com/ocxtal/libaw) alignment writer library.


## Format composition

The entire file consists of a single header line and multiple alignment lines. The field representation of the GPA format largely follows that of the Graphical Fragment Assembly (GFA) format, that is, each line starts with a single-character identifier, the fields are separated by tabs, and optional fields are distinguished by two-letter identifiers followed by a type character with a colon (:) between them.


### Field type identifier


* **A** single printable ASCII character
* **i** signed integer
* **Z** ASCII string (spaces allowed)

The other types found in the original GFA format, **f**, **H**, and **B** are reserved but not used in the current version.

### Optional field identifier

Each record has a optional fields at the tail of the line, separated with tabs. Each optional filed consists of a tuple of a tag, a type, and a value. The tag is a two-letter identifier followed by a type identifier character and content of the field delimited with colons.

### Header line

The header line is identified with a `H` character. The line have no mandatory field other than the line identifier and an optional version number field (`VN`).

#### Mandatory field

* **identifier:A** must be `'H'`

#### Optional field

* **VN:Z** string of the program version number

(is **SO:Z** (sorting order) required?)


#### Example

```
H	VN:Z:0.1
```

### Alignment line

The alignment line is identified with a 'A' character, having 10 mandatory fields to represent an alignment path and its projection (positions and lengths) to the two sequence segments.

#### Mandatory fields

* **identifier:A** must be `'A'`
* **name:Z** unique name of the alignment (string w/spaces)
* **aname:Z** name of the sequence segment A
* **apos:i** segment-local index (0-origin) of the alignment start position on the sequence segment A
* **alen:i** length of the alignment on the sequence segment A
* **adir:A** direction of the alignment, `'+'`(forward) or `'-'`(reverse)
* **bname:Z** name of the sequence segment B
* **bpos:i** segment-local index of the alignment start position on the sequence segment B
* **blen:i** length of the alignment on the sequence segment B
* **bdir:A** direction of the alignment on the sequence segment B
* **cigar:Z** CIGAR string of the alignment
* **prev** name of the previous (upstream) alignment
* **next** name of the next (downstream) alignment

#### Optional fields

* **MQ:i** mapping quality

#### Example

```
A	0	sec0	0	4	+	sec1	0	4	+	4M	*	*	MQ:i:255
```


## Example of a complete file

The example below represents a set of mappings of three sequences to a graphical reference structure with four sections. The read 1 aligns to the A->B->D path of the reference, 2 to the A->D, and 3 to the D->C->A (on the reverse-complemented side), respectively.

```
H	VN:Z:0.1
A	0	A	0	4	+	read1	0	4	+	4M	*	1	MQ:i:255
A	1	B	0	4	+	read1	4	4	+	4M	0	2	MQ:i:255
A	2	D	0	8	+	read1	8	8	+	8M	1	*	MQ:i:255
A	3	A	0	4	+	read2	0	4	+	4M	*	4	MQ:i:255
A	4	D	0	8	+	read2	4	8	+	8M	3	*	MQ:i:255
A	5	D	4	4	-	read3	16	4	+	4M	*	6	MQ:i:255
A	6	C	12	12	-	read3	4	12	+	12M	5	7	MQ:i:255
A	7	A	4	4	-	read3	0	4	+	4M	6	*	MQ:i:255
```
