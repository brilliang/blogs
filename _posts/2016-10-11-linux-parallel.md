---
layout: researcher
title: parallel your linux jobs
tags: linux, parallel
---
# {{ page.title }}

Linux tools have so beautiful abstraction of your daily work, that would not be easy to ignore them totally and to change to python or other script languages.

# when there are a lot of small files

Sometimes you want to accelerate your scripts by submit your jobs parallel. For example, I have 500 log files, which are all gz compressed. I want to uncompress them and grep some key words.

The most straightforward method you would think about is to submit them running in the background:

```bash
for i in $( find . -name \*.gz ); do gzip -d $i &; done
```

but if there are too many gz files, the method could grab all the computation resouce in that machine and possible trigger too many CPU swapping. We need a queue for these bash jobs.

parallel can easily handle this problem and even the grammar of the job becomes easier.

```bash
parallel gzip -d ::: $( find . -name \*.gz )
```

## when there is only one BIG file

parallel provide a method to distribut your data from pipe to several job executors.

```bash
cat bigfile.txt | parallel --block 10M --pipe grep 'pattern'

cat rands20M.txt | parallel --pipe awk \'{s+=\$1} END {print s}\' | awk '{s+=$1} END {print s}'

cat bigfile.txt | parallel  --pipe wc -l | awk '{s+=$1} END {print s}'

cat bigfile.txt | parallel --pipe sed s^old^new^g

```

However, someone challenges that the IO of cat function would be the bottleneck. I agree with it.

reference:
[Use multiple CPU Cores with your Linux commands — awk, sed, bzip2, grep, wc, etc.](http://www.rankfocus.com/use-cpu-cores-linux-commands/)
