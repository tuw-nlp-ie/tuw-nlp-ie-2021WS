# Text processing on the Linux command line

### Data-oriented Programming Paradigms,  2021 WS
10/12/2021

Gábor Recski

In this notebook we learn how to perform simple text processing tasks by combining a set of tools available on __UNIX-like systems__ (such as Linux and Mac OS) using __pipes__.

For a very brief introduction to the Linux command line, including links to additional documentation, see this notebook:


[Introduction to the Linux command line](https://github.com/tuw-nlp-ie/tuw-nlp-ie-2021WS/blob/main/lectures/01_Text_processing/01a_Intro_to_Linux_command_line.ipynb)

To run the commands inside this notebook, you need to install the bash kernel for jupyter, e.g. like this:
```
pip install bash-kernel
python -m bash_kernel.install
```


```bash
export LC_ALL=C
```

The following steps are based on two files in the `data` folder, `alice_tok.txt` contains the **tokenized** version of the novel _Alice in Wonderland_ and `data/stopwords.txt` contains a list of English **stopwords**, words that express some grammatical function that we often want to ignore in text processing applications. Both files were created in [this notebook](https://github.com/tuw-nlp-ie/tuw-nlp-ie-2021WS/blob/main/lectures/01_Text_processing/01_Text_processing.ipynb) on the basics of text processsing.

Let's have a look at the two files we are going to work with:


```bash
head data/alice_tok.txt
```


```bash
head data/stopwords.txt
```

`grep` is the command for matching regular expressions. Let's use it to find capitalized words.

The pipe symbol `|` means that the output of one command becomes the input of the next one:


```bash
grep -E '^[A-Z][a-z]+' data/alice_tok.txt | head
```

_You can ignore the occasional `Broken pipe` errors, it just means that a command in the pipeline was still writing output when the next one was already finished_

The `cat` command is often used at the beginning of a pipe, since all it does by default is send the contents of the file to the standard output


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | head
```

Counting can be implemented as a combination of sorting and aggregation:


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -20
```

Let's save this for later. The `sed` command can be used for regex-based search-and-replace, here we use it to get a more convenient format. Then we sort the lines alphabetically, later we'll see why.


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | sed 's/^ *\([0-9]*\) \(.*\)$/\2\t\1/' | sort | head
```


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | sed 's/^ *\([0-9]*\) \(.*\)$/\2\t\1/' | sort > data/ent_freqs.txt
```

Let's filter stopwords


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -50 | sed 's/^[ 0-9]*//' | head
```


```bash
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -50 | sed 's/^[ 0-9]*//' | sort | tr [:upper:] [:lower:] | comm -13 data/stopwords.txt - | head
```

Now let's find a way to get rid of the first words of sentences.


```bash
cat data/alice_tok.txt | sed 's/^$/@/' | tr '\n' ' ' | sed 's/ @ /\n/g' | cut -d' ' -f2- | tr ' ' '\n' | head
```


```bash
cat data/alice_tok.txt | sed 's/^$/@/' | tr '\n' ' ' | sed 's/ @ /\n/g' | cut -d' ' -f2- | tr ' ' '\n' | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -50 | sed 's/^[ 0-9]*//' | sort | tr [:upper:] [:lower:] | comm -13 data/stopwords.txt - | head -30
```

Now we can add the frequencies from the saved file


```bash
cat data/alice_tok.txt | sed 's/^$/@/' | tr '\n' ' ' | sed 's/ @ /\n/g' | cut -d' ' -f2- | tr ' ' '\n' | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -50 | sed 's/^[ 0-9]*//' | sort | tr [:upper:] [:lower:] | comm -13 data/stopwords.txt - | sed 's/^./\u&/' | join - data/ent_freqs.txt | sort -k2 -nr
```

### Homework

1. (25 points) Redo all steps of extracting a frequency count of entities, but using _Alice in Wonderland_ in another language. The German version is in `data/alice_de.txt`, but you can choose any other language for which you can find a plain text version online (try [Project Gutenberg](https://www.gutenberg.org/))! Start by adapting the preprocessing and segmentation steps in [this notebook](https://github.com/tuw-nlp-ie/tuw-nlp-ie-2021WS/blob/main/lectures/01_Text_processing/01_Text_processing.ipynb) to your chosen language and creating the two files used in this notebook (the tokenized text and the list of stopwords). Then check if the remaining steps also need modification.

1. (75 points) Improve the solution in this notebook to also include multi-word entities (we didn't find the Mock Turtle or the March Hare!). There may be many different ways to do this.

### Submission instructions

Submit your solution via TUWEL, by uploading 3 files:
- The tokenized input text for your chosen language (e.g. `alice_de_tok.txt`)
- The list of stopwords for your chosen language (e.g. `stopwords_de.txt`)
- A file with the extension `.sh` containing your command(s), with short explanations as comments (lines preceded by #).

__The cell below shows how the solution in this notebook would have to be submitted.__


```bash
# extract capitalized words, count them, reformat, save to file: 
cat data/alice_tok.txt | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | sed 's/^ *\([0-9]*\) \(.*\)$/\2\t\1/' | sort > data/ent_freqs.txt

# reformat to get one sentence per line, keep all but the first words of sentences, reformat again to one word per line, extract capitalized words, count them, keep the top 50, and then only those that are not in the stopwords file, finally match the lines to the lines of the word frequency file and sort by frequency
cat data/alice_tok.txt | sed 's/^$/@/' | tr '\n' ' ' | sed 's/ @ /\n/g' | cut -d' ' -f2- | tr ' ' '\n' | grep -E '^[A-Z][a-z]+' | sort | uniq -c | sort -nr | head -50 | sed 's/^[ 0-9]*//' | sort | tr [:upper:] [:lower:] | comm -13 data/stopwords.txt - | sed 's/^./\u&/' | join - data/ent_freqs.txt | sort -k2 -nr
```
