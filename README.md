# Zen Flesh, Zen Bones<!-- omit from toc -->

This repository holds code dedicated to extracting some knowledge graph our of the 1957 pdf version of Paul Reps' compilation
[Zen flesh, zen bones](./original_data/1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com.pdf).

## Table of contents<!-- omit from toc -->

- [Notes concerning the original book](#notes-concerning-the-original-book)
- [Running the converter](#running-the-converter)
- [Running the PDF conversion with docker](#running-the-pdf-conversion-with-docker)
- [Running the full data workflow](#running-the-full-data-workflow)
- [Development](#development)

## Notes concerning the original book

This directory holds a copy of the 1957 pdf version of Paul Reps' compilation
[Zen flesh, zen bones](./original_data/1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com.pdf) as offered online by [OceanoPDF](https://oceanofpdf.com/authors/paul-reps/pdf-epub-zen-flesh-zen-bones-a-collection-of-zen-and-pre-zen-writings-download/).

### References

- [ISBN-13: 9780385081030](https://isbnsearch.org/isbn/9780385081030)
- [Wikipedia](https://en.wikipedia.org/wiki/Zen_Flesh,_Zen_Bones)

### Book peculiarities

- Within chapter `101 ZEN STORIES`,
  - there are two sub-chapters with number `46.`
  - there is no sub-chapter numbered `49.`
- Within chapter the `CENTERING` chapter, there is a numbered list entry labeled `65. blah-blah without any a or m` that misses its ending dot character `.`. Alas such an entry matches the chapter pattern which fools the documentation reconstruction into believing this is a real chapter...
- They are many typos in the original text e.g. look for occurrences of `yon` in place of `you` in chapter `2. Finding a Diamond on a Muddy Road`... Refer to `typo_and_fix` entries within [`Convert/StructuralInfo.py`](./Convert/StructuralInfo.py).

## Running the converter

```bash
cd `git rev-parse --show-toplevel`/Convert
python3.10 -m venv venv
source ./venv/bin/activate
pip install -r requirements.txt
python main.py
```

## Running the PDF conversion with docker

```bash
docker build -t jejuness:doc_Zen_Flesh_Zen_Bones https://github.com/EricBoix/jj_doc_Zen_Flesh_Zen_Bones.git#:DockerContext
docker run --rm jejuness:doc_Zen_Flesh_Zen_Bones --help
```

Extracting the result out of the container requires local filesystem mount

```bash
docker run --rm  -v `pwd`/junk:/output jejuness:doc_Zen_Flesh_Zen_Bones --output_directory /output
```

## Running the full data workflow

Note: for a commented version of the following workflow refer e.g. to [the Four Noble Truth workflow](https://github.com/EricBoix/jj_doc_Four_Noble_Truths/blob/main/README.md#running-the-full-default-data-workflow).

Setup and context clean-up

```bash
cd `git rev-parse --show-toplevel`         # Implicit from now on
git clone https://github.com/EricBoix/jj_workflow_shell.git
source jj_workflow_shell/init.bash
```

```bash
export RESULTS_DIR=`pwd`/result_data       # Syntactic sugar
\rm -fr $RESULTS_DIR/database               # Clean slate from previous run
```

From original PDF to markdown and JSON

```bash
cd `git rev-parse --show-toplevel`
docker build -t jejuness:doc_Zen_Flesh_Zen_Bones https://github.com/EricBoix/jj_doc_Zen_Flesh_Zen_Bones.git#:DockerContext
docker run --rm  -v $RESULTS_DIR:/output jejuness:doc_Zen_Flesh_Zen_Bones --output_directory /output
```

Copy the `env-reference` file to a new `.env` file and customize the environment variables values in order to suit your needs

Prerequisite to Knowledge Graph (KG) extraction: launch a neo4j database

```bash
jj_launch_neo4j_db $RESULTS_DIR $NEO4J_PORT $NEO4J_USERNAME/$NEO4J_PASSWORD
```

Run the (Knowledge Graph) extraction

```bash
jj_extract_knowledge_graph $RESULTS_DIR '--load_markdown_document 1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com_-_local_converter.md  --load_json_document 1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com_-_Sentences_as_LangChain_Document.json'
```

Dump the database content for later usage (optional)

```bash
jj_dump_database $RESULTS_DIR neo4j.ZenFleshZenBones.MarkdownTextSplitterAndSentences.dump
```

In order to validate the dump, erase the database and restore it (out of the
previous dump)...

```bash
# WARNING: this DELETEs the existing database
rm -fr $RESULTS_DIR/database     
jj_restore_database $RESULTS_DIR neo4j.ZenFleshZenBones.MarkdownTextSplitterAndSentences.dump
jj_launch_neo4j_db $RESULTS_DIR $NEO4J_PORT $NEO4J_USERNAME/$NEO4J_PASSWORD
```

Extract knowledge graph in [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) format:

```bash
jj_dump_knowledge_graph_in_turtle $RESULTS_DIR ZenFleshZenBones.MarkdownTextSplitterAndSentences.ttl
```

Eventually turn the context off:

```bash
jj_stop_neo4j_db
```

## Development

### Testing

Within the above running context (directory and installed virtual environment)

```bash
cd `git rev-parse --show-toplevel`/Convert
pip install -r requirements-dev.txt
pytest test_main.py
```

### Updating/overwriting the result_data contents

Once development has improved some resulting converted files the following command will overwrite the reference resulting data

```bash
python main.py --output_directory ../result_data/
```
