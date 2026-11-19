sgRNA–Barcode Screening Analysis
This repository contains code for processing sequencing data from sgRNA–barcode screening experiments. The workflow extracts sgRNA and random DNA barcode sequences from FASTQ reads, associates them by sequencing read identifiers, and clusters the resulting barcode–sgRNA combinations using Bartender.

A small example FASTQ file, KOBCvaridation.fastq, is provided as a mini dataset (a subset of the original sequencing data) for testing the workflow. It is not the complete sequencing dataset.

Related preprint: https://www.biorxiv.org/content/10.1101/2025.02.23.639689v2


Part I — Repository contents
File	Description
screeningvector.txt	Workflow note containing Bartender commands and Python code for barcode extraction, read-identifier matching, and sequence clustering. It is not a single executable script.
KOBCvaridation.fastq	Mini dataset containing a subset of the original FASTQ reads, intended for testing the analysis workflow.
README.md	Documentation of the workflow, dependencies, inputs, and outputs.


Data availability note: The mini dataset is intended to be included in this repository. If KOBCvaridation.fastq is not visible in the repository file listing, it must be uploaded before the example commands can be run with this input.
Part II — System requirements
Software dependencies
Component	Required software/library	Version used or checked
sgRNA and DNA barcode extraction	Bartender (bartender_extractor_com)	Original version not confirmed
Sequence clustering	Bartender (bartender_single_com)	Original version not confirmed
Matching extracted sequences	Python (standard library)	Python 3.13.5 available in the checked Anaconda base environment; original analysis version not confirmed
Command-line execution	Bash or another compatible shell	GNU Bash 3.2.57(1)-release in the checked environment


The Python code uses only standard-library functionality; no additional Python packages are required for the sequence-association step.
Operating system and environment
The following environment was checked for documentation purposes:
- Operating system: macOS Sonoma 14.6.1 (build 23G93)
- Python: 3.13.5 (Anaconda base environment)
- GNU Bash: 3.2.57(1)-release
These are the available environment details, not confirmation that the complete pipeline was originally run or independently tested with all these versions. Bartender's version and any other operating systems used have not been documented. A separate test environment is not required simply because it exists; list it only if it was actually used for this workflow and is necessary for reproducibility.
Hardware and runtime
Hardware specifications, memory requirements, and typical execution times have not been established. Runtime and storage use depend on the FASTQ size and number of unique barcode sequences. The provided mini dataset is intended to make an initial test less resource-intensive than processing the complete dataset.


Part III — sgRNA and DNA barcode processing
The following steps reproduce the processing logic in screeningvector.txt, with one filename inconsistency corrected as indicated below. Run the shell commands in a directory containing KOBCvaridation.fastq, or replace that filename with the path to another input FASTQ file.
Step 1 — Prepare the FASTQ input
The example input is:
KOBCvaridation.fastq
This is a mini dataset, not the full sequencing dataset. The same sequence-extraction commands can be applied to the complete original FASTQ dataset by changing the input file path. Input reads must conform to the sequence patterns used below.
Step 2 — Extract sgRNA sequences
bartender_extractor_com \
  -f KOBCvaridation.fastq \
  -o KOBCsgRNA \
  -q '?' \
  -p 'CACCG[20]GTTTA' \
  -m 2
This command extracts a 20-nucleotide sgRNA sequence between the specified flanking sequences. The extraction pattern and -m 2 setting are those in the original workflow. The expected extracted-sequence file used by the next step is KOBCsgRNA_barcode.txt.
Step 3 — Extract random DNA barcodes
bartender_extractor_com \
  -f KOBCvaridation.fastq \
  -o KOBCvaridation \
  -q ':' \
  -p 'ATTAA[25]GGCCT' \
  -m 2
This command extracts a 25-nucleotide random DNA barcode between the specified flanking sequences. The expected extracted-sequence file used by the next step is KOBCvaridation_barcode.txt.

The -q characters and extraction parameters are preserved from screeningvector.txt; confirm that they are appropriate for the input read design and installed Bartender version.
Step 4 — Match sgRNAs and random DNA barcodes by read identifier
The original Python logic matches records by the identifier in the second comma-separated field of the two extracted-sequence files. It concatenates the 25-nt random DNA barcode first, followed by the 20-nt sgRNA, to produce a 45-nt combined sequence.

Run the following as a Python script or in an interactive Python session (not directly as shell commands):
def merge_sequences_with_numbers(file1, file2, output_file):
    sequences = {}
    with open(file1, "r") as f1:
        for line in f1:
            sequence, number = line.strip().split(",")
            sequences[number] = sequence

    with open(file2, "r") as f2, open(output_file, "w") as output:
        for line in f2:
            sequence, number = line.strip().split(",")
            if number in sequences:
                merged_sequence = sequences[number] + sequence
                output.write(f"{merged_sequence},{number}\n")

    print(f"Merged sequences saved in {output_file}.")


merge_sequences_with_numbers(
    "KOBCvaridation_barcode.txt",
    "KOBCsgRNA_barcode.txt",
    "merged_sequences.txt",
)
The output is merged_sequences.txt, with one matching barcode–sgRNA combination and its read identifier per line. This code assumes that input lines have exactly two comma-separated fields, sequence,identifier; it does not perform additional quality filtering or verify one-to-one matches among repeated identifiers.
Step 5 — Cluster combined sequences with Bartender
bartender_single_com \
  -f merged_sequences.txt \
  -o KOBCsgRNA \
  -d 3
The original workflow uses an edit-distance threshold of 3 (-d 3). The command above uses merged_sequences.txt to match the actual filename produced by the Python code.

Filename correction: The final command in the original screeningvector.txt refers to merged_sequence.txt (singular), whereas its Python code writes merged_sequences.txt (plural). The command above corrects that mismatch. If required by the installed Bartender version, confirm or adapt the input format of the merged file before running clustering.


Part IV — Output
Intermediate and final files
File/output	Description
KOBCsgRNA_barcode.txt	Extracted 20-nt sgRNA sequences with read identifiers (expected Bartender extraction output).
KOBCvaridation_barcode.txt	Extracted 25-nt random DNA barcodes with read identifiers (expected Bartender extraction output).
merged_sequences.txt	Concatenated 25-nt DNA barcode + 20-nt sgRNA sequences, associated with read identifiers.
Bartender clustering output with prefix KOBCsgRNA	Cluster assignments and sequence-count information, according to the installed Bartender version and its output format.



Part V — Citation
Please cite the associated study when using these materials:

https://www.biorxiv.org/content/10.1101/2025.02.23.639689v2
