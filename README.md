# Zen Flesh, Zen Bones<!-- omit from toc -->

This repository holds code dedicated to extracting some knowledge graph our of the 1957 pdf version of Paul Reps' compilation
[Zen flesh, zen bones](./original_data/1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com.pdf).

## Table of contents<!-- omit from toc -->

- [Notes concerning the original book](#notes-concerning-the-original-book)
- [Running the converter](#running-the-converter)
- [Running the full data workflow with jejune\_cli](#running-the-full-data-workflow-with-jejune_cli)

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

### Running with local installation

```bash
cd `git rev-parse --show-toplevel`/Convert
python3.10 -m venv venv
source ./venv/bin/activate
pip install -r requirements.txt
python main.py
```

### Testing the conversion results

Within the above running context (directory and installed virtual environment)

```bash
cd `git rev-parse --show-toplevel`/Convert
pip install -r requirements-dev.txt
pytest test_main.py
```

### Developers: updating/overwriting the result_data contents

Once development has improved some resulting converted files the following command will overwrite the reference resulting data

```bash
python main.py --output_directory ../result_data/
```

### Running the PDF conversion with docker

```bash
docker build -t jejune:doc_Zen_Flesh_Zen_Bones https://github.com/EricBoix/jj_doc_Zen_Flesh_Zen_Bones.git#:DockerContext
docker run --rm jejune:doc_Zen_Flesh_Zen_Bones --help
```

Extracting the result out of the container requires local filesystem mount

```bash
docker run --rm  -v `pwd`/junk:/output jejune:doc_Zen_Flesh_Zen_Bones --output_directory /output
```

## Running the full data workflow with jejune_cli

Install and configure [`jejune_cli`](https://github.com/EricBoix/jejune_cli), then run `jejune doctor` to verify the configuration. This boils down to

```bash
uv tool install git+https://github.com/EricBoix/jejune_cli
jejune configuration init     
# Proceed with the configuration of the files located in .jejune/ sub-directory.
# Assert the configuration is sound with
jejune doctor
```

Define a convenience variable for the results directory:

```bash
export RESULTS_DIR=`pwd`/result_data
```

Run the converter to extract a markdown out of the original PDF :

```bash
jejune convert build
jejune convert run --output-dir $RESULTS_DIR
```

Run the (Knowledge Graph) extraction (starting a neo4j database being prerequisite)

```bash
jejune neo4j delete $RESULTS_DIR    # Avoid collision with previous/other run
jejune neo4j stats --assert 0/0     # Just making sure deletion was effective
jejune neo4j start $RESULTS_DIR
jejune graph extract $RESULTS_DIR \
  --load_markdown_document \
    1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com_-_local_converter.md \
  --load_json_document \
    1957_-_Paul_Reps_-_Zen_flesh_zen_bones-A_Collection_of_Zen_and_Pre_Zen_Writings_-_Scan_by_OceanofPDF_dot_com_-_Sentences_as_LangChain_Document.json
jejune neo4j stats --assert 4769/9962
```

Optional: dump the database content for later usage (and restore it to assert dump integrity/validity)

```bash
jejune neo4j stop
jejune neo4j dump $RESULTS_DIR neo4j.ZenFleshZenBones.MarkdownTextSplitterAndSentences.dump
# Restore the database out of the dump (just to make sure)
# WARNING: restoring DELETEs the existing database
jejune neo4j restore $RESULTS_DIR neo4j.ZenFleshZenBones.MarkdownTextSplitterAndSentences.dump
jejune neo4j start $RESULTS_DIR
jejune neo4j stats --assert 4769/9962
```

Extract knowledge graph in [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) format

```bash
jejune neo4j dump-turtle $RESULTS_DIR ZenFleshZenBones.MarkdownTextSplitterAndSentences.ttl
jejune neo4j stop
```
