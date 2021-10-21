# Introduction to the Linux command line

### Data-oriented Programming Paradigms,  2021 WS
10/12/2021

Gábor Recski

There's a huge amount of documentation on Linux programming (start e.g. [here](https://tldp.org/index.html)), this is an introduction to only a very small and specific subset.

The commands below are typically run in a terminal, which is actually the interpreter of a [shell](), such as [bash](), the default shell on most Linux distributions.

To run the commands inside this notebook, you need to install the bash kernel for jupyter, e.g. like this:
```
pip install bash-kernel
python -m bash_kernel.install
```


```bash
export LC_ALL=C
```

The first command sets an __environment variable__ to control the system __locale__ for this session, this ensures that your language and region settings do not interfere with how some tools such as __sort__ work in the examples below. __You need not understand this for now__, but these are still topics worth getting familiar with:
- [locale](https://en.wikipedia.org/wiki/Locale_(computer_software))
- [environment variables](https://en.wikipedia.org/wiki/Environment_variable)
- [sorting and locale](https://stackoverflow.com/questions/28881/why-doesnt-sort-sort-the-same-on-every-machine).

Let's look at the contents of the data directory


```bash
ls -l data
```

`ls` is the name of a program for listing directory contents, `-l` is an **option**. Most programs used in this notebook have dozens of options and you shouldn't expect to learn about all of them at once.

You can get a summary of what each command does and what paramteres it has using the `man` command


```bash
man ls
```

For example, this command will list the directory contents by file size and also use human-readable numbers for file size:


```bash
ls -lSh data
```

### Day-to-day tasks with pipes

##### What jupyter processes am I running?


```bash
ps aux | grep `whoami` | grep jupyter
```

##### What directories are using up all the disk space?


```bash
du -h --max-depth=1 | sort -h
```

### Simple data processing with pipes

The file in `data/ta_restaurants_31EU.tsv` contains basic information about restaurants in 31 EU cities, from TripAdvisor. It is based on a [Kaggle dataset](https://www.kaggle.com/damienbeneschi/krakow-ta-restaurans-data-raw).

#### What is in the data?


```bash
cat data/ta_restaurants_31EU.tsv | head
```

#### How many restaurants? How many in Vienna? How many in each city?


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | wc -l
```


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | grep 'Vienna' | wc -l
```


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | cut -f2 | sort | uniq -c | sort -nr
```

#### What are the top cuisines in the dataset?


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | cut -f3 | tr '|' '\n' | sort | uniq -c | sort -nr
```

#### Which are the top-rated restaurants in Vienna?


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | awk -F$'\t' '$2 == "Vienna"' | sort -t $'\t' -k4 -gr | head
```

#### OK, how about top-rated cheap restaurants?


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | awk -F$'\t' '$2 == "Vienna"'  | grep -v '\$\$' | sort -t $'\t' -k4 -gr | head
```

#### How about top-rated, cheap, vegan restaurants?


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | awk -F$'\t' '$2 == "Vienna"'  |  grep '\$' | grep -v '\$\$' | grep 'Vegan Options' | sort -t $'\t' -k4 -gr | head
```

#### Finally, filter those with less than 100 reviews


```bash
cat data/ta_restaurants_31EU.tsv | grep -v '^#' | awk -F$'\t' '$2 == "Vienna"'  |  grep '\$' | grep -v '\$\$' | grep 'Vegan Options' | awk -F$'\t' '$6 >= 100' | sort -t $'\t' -k4 -gr 
```
