---
title: "Counting and mining with the shell"
teaching: 40
exercises: 20
questions:
- "How can I count data?"
- "How can I find data within files?"
- "How can I combine existing commands to do new things?"
objectives:
- "Understand how to count lines, words, and characters with the shell"
- "Understand how to mine files and extract matched lines with the shell"
- "Understand how to combine mining with the shell and regular expressions"
- "Redirect a command’s output to a file."
- "Process a file instead of keyboard input using redirection."
- "Construct command pipelines with two or more stages."
- "Explain Unix’s ‘small pieces, loosely joined’ philosophy."
keypoints:
- "`wc` is a command that counts"
- "use the `wc` command with the flags `-w` and `-l` to count the words and lines in a file or a series of files"
- "use the redirector and structure `> subdirectory/filename` to save results into a subdirectory"
- "use the `grep` command to search for instances of a string inside files"
- "use `grep` with the `-c` flag to count instances of a string, the `-i` flag to return a case insensitive search for a string, the `-v` flag to exclude a string from the results, and `-w` to return a whole word only search"
- "use `--file=list.txt` to use the file `list.txt` as the source of strings used in a query"
- "combine these commands and flags to build complex queries in a way that suggests the potential for using the Unix shell to count and mine your research data and research projects"

---
##  Counting and mining data

Now that you know a little bit about navigating the shell, we will move onto
learning how to count and mine data using a few of the standard shell commands.
While these commands are unlikely to revolutionise your work by themselves,
they're very versatile and will add to your foundation for working in the shell
and for learning to code.

## Counting and sorting

We will begin by counting the contents of files using the Unix shell.
We can use the Unix shell to quickly generate counts from across files,
something that is tricky to achieve using the graphical user interfaces of standard office suites.

Let's start by navigating to the directory that contains our data using the
`cd` command:

~~~
$ cd shell-lesson
~~~
{: .bash}

Remember, if at any time you are not sure where you are in your directory structure,
use the `pwd` command to find out:

~~~
$ pwd
~~~
{: .bash}
~~~
/Users/riley/Desktop/shell-lesson
~~~
{: .output}

And let's just check what files are in the directory and how large they
are with `ls -lh`:

~~~
$ ls -lh
~~~
{: .bash}
~~~
total 139M
-rw-r--r-- 1 riley staff 3.6M Jan 31 18:47 2014-01-31_JA-africa.tsv
-rw-r--r-- 1 riley staff 7.4M Jan 31 18:47 2014-01-31_JA-america.tsv
-rw-rw-r-- 1 riley staff 126M Jun 10  2015 2014-01_JA.tsv
-rw-r--r-- 1 riley staff 1.4M Jan 31 18:47 2014-02-02_JA-britain.tsv
-rw-r--r-- 1 riley staff 583K Feb  1 22:53 33504-0.txt
drwxr-xr-x 2 riley staff   68 Feb  2 00:58 backup
-rw-r--r-- 1 riley staff 598K Jan 31 18:47 gulliver.txt
-rw-r--r-- 1 riley staff   13 Jan 31 18:47 gallic.txt
~~~
{: .output}

In this episode we'll focus on the dataset `2014-01_JA.tsv`, that contains
journal article metadata, and the three `.tsv` files derived from the original
dataset. Each of these three .tsv files includes all data where a keyword such
as `africa` or `america` appears in the 'Title' field of `2014-01_JA.tsv`.

> ## CSV and TSV Files
> CSV (Comma-separated values) is a common plain text format for storing tabular
> data, where each record occupies one line and the values are separated by commas.
> TSV (Tab-separated values) is just the same except that values are separated by
> tabs rather than commas. Confusingly, CSV is sometimes used to refer to both CSV,
> TSV and variations of them. The simplicity of the formats make them great for
> exchange and archival. They are not bound to a specific program (unlike Excel
> files, say, there is no `CSV` program, just lots and lots of programs that
> support the format, including Excel by the way.), and you wouldn't have any
> problems opening a 40 year old file today if you came across one.
{: .callout}
<!-- hm, reminds me of MARC -->

`wc` is the "word count" command: it counts the number of lines, words, bytes
and characters in files. Since we love the wildcard operator, let's run the command
`wc *.tsv` to get counts for all the `.tsv` files in the current directory
(it takes a little time to complete):

~~~~
$ wc *.tsv
~~~~
{: .bash}
~~~
    13712    511261   3773660 2014-01-31_JA-africa.tsv
    27392   1049601   7731914 2014-01-31_JA-america.tsv
   507732  17606310 131122144 2014-01_JA.tsv
     5375    196999   1453418 2014-02-02_JA-britain.tsv
   554211  19364171 144081136 total
~~~
{: .output}

The first three columns contains the number of lines, words and bytes
(to show number characters you have to use a flag).

If we only have a handful of files to compare, it might be faster or more convenient
to just check with Microsoft Excel, OpenRefine or your favourite text editor, but
when we have tens, hundres or thousands of documents, the Unix shell has a clear
speed advantage. The real power of the shell comes from being able to combine commands
and automate tasks, though. We will touch upon this slightly.

For now, we'll see how we can build a simple pipeline to find the shortest file
in terms of number of lines. We start by adding the `-l` flag to get only the
number of lines, not the number of words and bytes:

~~~~
$ wc -l *.tsv
~~~~
{: .bash}
~~~
    13712 2014-01-31_JA-africa.tsv
    27392 2014-01-31_JA-america.tsv
   507732 2014-01_JA.tsv
     5375 2014-02-02_JA-britain.tsv
   554211 total
~~~
{: .output}

The `wc` command itself doesn't have a flag to sort the output, but as we'll
see, we can combine three differen shell commands to get what we want.

First, we have the `wc -l *.tsv` command. We will save the output from this
command in a new file. To do that, we *redirect* the output from the command
to a file using the ‘greater than’ sign (>), like so:

~~~
$ wc -l *.tsv > lengths.txt
~~~
{: .bash}

There's no output now since the output went into the file `lengths.txt`, but
we can check that the output indeed ended up in the file using `cat` or `less`
(or Notepad or any text editor).

~~~~
$ cat lengths.txt
~~~~
{: .bash}
~~~
    13712 2014-01-31_JA-africa.tsv
    27392 2014-01-31_JA-america.tsv
   507732 2014-01_JA.tsv
     5375 2014-02-02_JA-britain.tsv
   554211 total
~~~
{: .bash}

Next, there is the `sort` command. We'll use the `-n` flag to specify that we
want numerical sorting, not lexical sorting, we output the results into
yet another file, and we use `cat` to check the results:

~~~~
$ sort -n lengths.txt > sorted-lengths.txt
$ cat sorted-lengths.txt
~~~~
{: .bash}
~~~
     5375 2014-02-02_JA-britain.tsv
    13712 2014-01-31_JA-africa.tsv
    27392 2014-01-31_JA-america.tsv
   507732 2014-01_JA.tsv
   554211 total
~~~
{: .output}

Finally we have our old friend `head`, that we can use to get the first line
of the `sorted-lengths.txt`:

~~~~
$ head sorted-lengths.txt
~~~~
{: .bash}
~~~
     5375 2014-02-02_JA-britain.tsv
~~~
{: .output}

But we're really just interested in the end result, not the intermediate
results now stored in `lengths.txt` and `sorted-lengths.txt`. What if we could
send the results from the first command (`wc -l *.tsv`) directly to the next
command (`sort -n`) and then the output from that command to `head -n 1`?
Luckily we can, using a concept called pipes. On the command line, you make a
pipe with the vertical bar character |. Let's try with one pipe first:

~~~~
$ wc -l *.tsv | sort -n
~~~~
{: .bash}
~~~
     5375 2014-02-02_JA-britain.tsv
    13712 2014-01-31_JA-africa.tsv
    27392 2014-01-31_JA-america.tsv
   507732 2014-01_JA.tsv
   554211 total
~~~
{: .output}

Notice that this is exactly the same output that ended up in our `sorted-lengths.txt`
earlier. Let's add another pipe:

~~~~
$ wc -l *.tsv | sort -n | head -n 1
~~~~
{: .bash}
~~~
     5375 2014-02-02_JA-britain.tsv
~~~
{: .output}

It can take some time to fully grasp pipes and use them efficiently, but it's a
very powerful concept that you will find not only in the shell, but also in most
programming languages.

![Redirects and Pipes](../fig/redirects-and-pipes.png)

> ## Pipes and Filters
> This simple idea is why Unix has been so successful. Instead of creating enormous
> programs that try to do many different things, Unix programmers focus on creating
> lots of simple tools that each do one job well, and that work well with each other.
> This programming model is called “pipes and filters”. We’ve already seen pipes; a
> filter is a program like `wc` or `sort` that transforms a stream of input into a
> stream of output. Almost all of the standard Unix tools can work this way: unless
> told to do otherwise, they read from standard input, do something with what they’ve
> read, and write to standard output.
>
> The key is that any program that reads lines of text from standard input and writes
> lines of text to standard output can be combined with every other program that
> behaves this way as well. You can and should write your programs this way so that
> you and other people can put those programs into pipes to multiply their power.
{: .callout}
<!-- Copied from https://swcarpentry.github.io/shell-novice/04-pipefilter/ -->

> ## Adding another pipe
> We have our `wc -l *.tsv | sort -n | head -n 1` pipeline. What would happen
> if you piped this into `cat`? Try it!
>
> > ## Solution
> > The `cat` command just outputs whatever it gets as input, so you get exactly
> > the same output from
> >
> > ~~~
> > $ wc -l *.tsv | sort -n | head -n 1
> > ~~~
> > {: .bash}
> >
> > and
> >
> > ~~~
> > $ wc -l *.tsv | sort -n | head -n 1 | cat
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Counting number of files, part I
> Let's make a different pipeline. You want to find out how many files and
> directories there are in the current directory. Try to see if you can pipe
> the output from `ls` into `wc` to find the answer, or something close to the
> answer.
>
> > ## Solution
> > You get close with
> >
> > ~~~
> > $ ls -l | wc -l
> > ~~~~
> > {: .bash}
> >
> > but the count will be one too high, since the "total" line from `ls`
> > is included in the count. We'll get back to a way to fix that later
> > when we've learned about the `grep` command.
> {: .solution}
{: .challenge}

> ## Writing to files
> The `date` command outputs the current date and time. Can you write the
> current date and time to a new file called `logfile.txt`? Then check
> the contents of the file.
>
> > ## Solution
> > ~~~
> > $ date > logfile.txt
> > $ cat logfile.txt
> > ~~~~
> > {: .bash}
> > To check the contents, you could also use `less` or many other commands.
> >
> > Beware that `>` will happily overwrite an existing file without warning you,
> > so please be careful.
> {: .solution}
{: .challenge}

> ## Appending to a file
> While `>` writes to a file, `>>` appends something to a file. Try to append the
> current date and time to the file `logfile.txt`?
>
> > ## Solution
> > ~~~
> > $ date >> logfile.txt
> > $ cat logfile.txt
> > ~~~~
> > {: .bash}
> {: .solution}
{: .challenge}

## Mining or searching

Searching for something in one or more files is something we'll often need to do,
so let's introduce a command for doing that: `grep` (short for **global regular
expression print**). As the name suggests, it supports regular expressions and
is therefore only limited by your imagination, the shape of your data, and - when
working with thousands or millions of files - the processing power at your disposal.

To begin using `grep`, first navigate to the `data` directory (from results/ type `cd ..`).

~~~
$ grep 1999 *.tsv
~~~
{: .bash}

This query looks across all files in  the directory that fit the given criteria (the .tsv files) for instances of the string, or character cluster, '1999'. It then prints them within the shell.

Press the up arrow once in order to cycle back to your most recent action.
Amend `grep 1999 *.tsv` to `grep -c 1999 *.tsv` and hit enter.

~~~
$ grep -c 1999 *.tsv
~~~
{: .bash}
~~~
2014-01-31_JA-africa.tsv:804
2014-01-31_JA-america.tsv:1478
2014-01_JA.tsv:28767
2014-02-02_JA-britain.tsv:284
~~~
{: .output}

The shell now prints the number of times the string 1999 appeared in each `*.tsv file`.
If you look at the output from the previous command, this tends to be refer to the
date field for each journal article.

Strings need not be numbers.

~~~
$ grep -c revolution *.tsv
~~~
{: .bash}
~~~
2014-01-31_JA-africa.tsv:20
2014-01-31_JA-america.tsv:34
2014-01_JA.tsv:867
2014-02-02_JA-britain.tsv:9
~~~
{: .output}

Counts
the instances of the string `revolution` within the defined files and prints
those counts to the shell. Now, amend the above command to the below and observer how the outpu of each is different:

~~~
$ grep -ci revolution *.tsv
~~~
{: .bash}
~~~
2014-01-31_JA-africa.tsv:118
2014-01-31_JA-america.tsv:1018
2014-01_JA.tsv:9327
2014-02-02_JA-britain.tsv:122
~~~
{: .output}

This repeats the query, but prints a case
insensitive count (including instances of both `revolution` and `Revolution`).
Note how the count has increased nearly 30 fold for those journal article
titles that contain the keyword 'america'. As before, cycling back and
adding `> results/`, followed by a filename (ideally in .txt format), will save the results to a data file.

So far we have counted strings in file and printed to the shell or to
file those counts. But the real power of `grep` comes in that you can
also use it to create subsets of tabulated data (or indeed any data)
from one or multiple files.  

~~~
$ grep -i revolution *.tsv
~~~
{: .bash}

This script looks in the defined files and prints any lines containing `revolution`
(without regard to case) to the shell.

~~~
$ grep -i revolution *.tsv > results/2016-07-19_JAi-revolution.tsv
~~~
{: .bash}

This saves the subsetted data to file.

However if we look at this file, it contains every instance of the
string 'revolution' including as a single word and as part of other words
such as 'revolutionary'. This perhaps isn't as useful as we thought...
Thankfully, the `-w` flag instructs `grep` to look for whole words only,
giving us greater precision in our search.

~~~
$ grep -iw revolution *.tsv > results/DATE_JAiw-revolution.tsv
~~~
{: .bash}

This script looks in both of the defined files and
exports any lines containing the whole word `revolution` (without regard to case)
to the specified .tsv file.

We can show the difference between the files we created.

~~~
$ wc -l results/*.tsv
~~~
{: .bash}
~~~
   10695 2016-07-19_JAi-revolution.tsv
    7859 2016-07-19_JAw-revolution.tsv
   18554 total
~~~
{: .output}

Finally, you can use the **regular expression syntax** covered earlier to search for similar words.

In `gallic.txt` we have the string `fr[ae]nc[eh]`.

~~~
$ cat gallic.txt
~~~
{: .bash}
~~~
fr[ae]nc[eh]
~~~
{: .output}

The square brackets here ask the machine to match any character
in the range specified. So when used with grep:

~~~
$ grep -iw --file=gallic.txt *.tsv
~~~
{: .bash}

the shell will print out each line containing the string:

~~~
- france
- french
- frence
- franch
~~~
{: .output}

Include the `-o` flag to print only the matching part of the lines e.g. (handy for isolating/checking results).

~~~
$ grep -iwo revolution *.tsv
~~~
{: .bash}

OR:

~~~
$ grep -iwo --file=gallic.txt *.tsv
~~~
{: .bash}

Pair up with your neighbor and work on these exercies:

> ## Case sensitive search
> Search for all case sensitive instances of
> a word you choose in all four derived tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > grep hero *.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case sensitive search in select files
> Search for all case sensitive instances of a word you choose in
> the 'America' and 'Africa' tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > grep hero *a.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Count words (case sensitive)
> Count all case sensitive instances of a word you choose in
> the 'America' and 'Africa' tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > grep -c hero *a.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Count words (case insensitive)
> Count all case insensitive instances of that word in the 'America' and 'Africa' tsv files
> in this directory. Print your results to the shell.
>
> > ## Solution
> > ~~~
> > grep -ci hero *a.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case insensitive search in select files
> Search for all case insensitive instances of that
> word in the 'America' and 'Africa' tsv files in this directory. Print your results to a `new >.tsv` file.
>
> > ## Solution
> > ~~~
> > grep -i hero *a.tsv > new.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case insensitive search in select files (whole word)
> Search for all case insensitive instances of that whole word
> in the 'America' and 'Africa' tsv files in this directory. Print your results to a new.tsv > file.
>
> > ## Solution
> > ~~~
> > grep -iw hero *a.tsv > new2.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Counting number of files, part II
> In the earlier counting exercise in this episode, you tried counting the number
> of files and directories in the current directory.
> * The command `ls -l | wc -l` took us quite far, but the result was one too
>   high because it included the "total" line in the count.
> * With the knowledge of `grep`, can you figure out how to exclude the "total"
>   line from the `ls -l` output?
> * Hint: You want to exclude any line *starting*
>   with the text "total", and you've just learned that the hat character (^) is used
>   in regular expressions to indicate the start of a line.
>
> > ## Solution
> > To find any lines starting with "total", we would use:
> >
> > ~~~
> > $ ls -l | grep -E '^total'
> > ~~~
> > {: .bash}
> >
> > To *exclude those lines, we add the `-v` flag:
> >
> > ~~~
> > $ ls -l | grep -v -E '^total'
> > ~~~
> > {: .bash}
> >
> > The grand finale is to pipe this into `wc -l`:
> >
> > ~~~
> > $ ls -l | grep -v -E '^total | wc -l'
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

Compare the line counts of the last two files you created.

~~~
wc -l FILENAMES
~~~
{: .bash}

Open both files in a text editor (Notepad++, Atom, Kate,
whatever you prefer) or Excel-like program to see the difference
between searching strings and searching whole words using `grep`.
