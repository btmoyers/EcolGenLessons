---
title: "Redirection"
teaching: 35
exercises: 30
questions:
- "How can I search within files?"
- "How can I combine existing commands to do new things?"
objectives:
- "Employ the `grep` command to search for information within files."
- "Print the results of a command to a file."
- "Construct command pipelines with two or more stages."
keypoints:
- "`grep` is a powerful search tool with many options for customization."
- "`>`, `>>`, and `|` are different ways of redirecting output."
- "`command > file` redirects a command's output to a file."
- "`command >> file` redirects a command's output to a file without overwriting the existing contents of the file."
- "`command_1 | command_2` redirects the output of the first command as input to the second command."
---

All of the exercises from this section on will be run on the remote server (with one exception--can you guess which one that might be?). Log on to the MGHPCC using `ssh your-user-name@ghpcc06.umassrc.org`.

> ## Exercise
> 
> Use what you know so far to delete the single fastq file from your home directory on the MGHPCC and securely copy the whole `shell_data/` directory there instead. To move the whole directory and its contents at once you're going to need to look at the options for `scp`. Remember that applying the same command to a directory and everything that it contains is called 'recursive'.
> 
> > ## Solution
> > `scp -r`, for recursive, will secure copy a directory and everything it contains. So to move the whole `shell_data/` directory at once, you would type `scp -r /path/to/shell_data/ your-user-name@ghpcc06.umassrc.org:/home/your-user-name/`.
> {: .solution}
{: .challenge}


## Searching files

We discussed in a previous episode how to search within a file using `less`. We can also
search within files without even opening them, using `grep`. `grep` is a command-line
utility for searching plain-text files for lines matching a specific set of 
characters (sometimes called a string) or a particular pattern 
(which can be specified using something called regular expressions). We're not going to work with 
regular expressions in this lesson, and are instead going to specify the strings 
we are searching for.
Let's give it a try!

> ## Nucleotide abbreviations
> 
> The four nucleotides that appear in DNA are abbreviated `A`, `C`, `T` and `G`. 
> Unknown nucleotides are represented with the letter `N`. An `N` appearing
> in a sequencing file represents a position where the sequencing machine was not able to 
> confidently determine the nucleotide in that position. You can think of an `N` as being aNy 
> nucleotide at that position in the DNA sequence. 
> 
{: .callout}

We'll search for strings inside of our fastq files. Let's first make sure we are in the correct 
directory:

~~~
$ cd ~/shell_data/untrimmed_fastq
~~~
{: .bash}

Suppose we want to see how many reads in our file have really bad segments containing 10 consecutive unknown nucleotides (Ns).

> ## Determining quality
> 
> In this lesson, we're going to be manually searching for strings of `N`s within our sequence
> results to illustrate some principles of file searching. It can be really useful to do this
> type of searching to get a feel for the quality of your sequencing results, however, in your 
> research you will most likely use a bioinformatics tool that has a built-in program for
> filtering out low-quality reads. You'll learn how to use one such tool in 
> [a later lesson](https://datacarpentry.org/wrangling-genomics/02-quality-control/index.html).
> 
{: .callout}

Let's search for the string NNNNNNNNNN in the SRR098026 file:
~~~
$ grep NNNNNNNNNN SRR098026.fastq
~~~
{: .bash}

This command returns a lot of output to the terminal. Every single line in the SRR098026 
file that contains at least 10 consecutive Ns is printed to the terminal, regardless of how long or short the file is. 
We may be interested not only in the actual sequence which contains this string, but 
in the name (or identifier) of that sequence. We discussed in a previous lesson 
that the identifier line immediately precedes the nucleotide sequence for each read
in a FASTQ file. We may also want to inspect the quality scores associated with
each of these reads. To get all of this information, we will return the line 
immediately before each match and the two lines immediately after each match.

We can use the `-B` argument for grep to return a specific number of lines before
each match. The `-A` argument returns a specific number of lines after each matching line. Here we want the line *before* and the two lines *after* each 
matching line, so we add `-B1 -A2` to our grep command:

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq
~~~
{: .bash}

One of the sets of lines returned by this command is: 

~~~
@SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
CNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
+SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
~~~
{: .output}

> ## Exercise
>
> 1. Search for the sequence `GNATNACCACTTCC` in the `SRR098026.fastq` file.
> Have your search return all matching lines and the name (or identifier) for each sequence
> that contains a match.
> 
> 2. Search for the sequence `AAGTT` in both FASTQ files.
> Have your search return all matching lines and the name (or identifier) for each sequence
> that contains a match.
> 
> > ## Solution  
> > 1. `grep -B1 GNATNACCACTTCC SRR098026.fastq` 
> > 
> >     ```
> >     @SRR098026.245 HWUSI-EAS1599_1:2:1:2:801 length=35
> >     GNATNACCACTTCCAGTGCTGANNNNNNNGGGATG
> >     ```
> > 
> > 2. `grep -B1 AAGTT *.fastq`
> >
> >     ```
> > SRR097977.fastq-@SRR097977.11 209DTAAXX_Lenski2_1_7:8:3:247:351 length=36
> > SRR097977.fastq:GATTGCTTTAATGAAAAAGTCATATAAGTTGCCATG
> > --
> > SRR097977.fastq-@SRR097977.67 209DTAAXX_Lenski2_1_7:8:3:544:566 length=36
> > SRR097977.fastq:TTGTCCACGCTTTTCTATGTAAAGTTTATTTGCTTT
> > --
> > SRR097977.fastq-@SRR097977.68 209DTAAXX_Lenski2_1_7:8:3:724:110 length=36
> > SRR097977.fastq:TGAAGCCTGCTTTTTTATACTAAGTTTGCATTATAA
> > --
> > SRR097977.fastq-@SRR097977.80 209DTAAXX_Lenski2_1_7:8:3:258:281 length=36
> > SRR097977.fastq:GTGGCGCTGCTGCATAAGTTGGGTTATCAGGTCGTT
> > --
> > SRR097977.fastq-@SRR097977.92 209DTAAXX_Lenski2_1_7:8:3:353:318 length=36
> > SRR097977.fastq:GGCAAAATGGTCCTCCAGCCAGGCCAGAAGCAAGTT
> > --
> > SRR097977.fastq-@SRR097977.139 209DTAAXX_Lenski2_1_7:8:3:703:655 length=36
> > SRR097977.fastq:TTTATTTGTAAAGTTTTGTTGAAATAAGGGTTGTAA
> > --
> > SRR097977.fastq-@SRR097977.238 209DTAAXX_Lenski2_1_7:8:3:592:919 length=36
> > SRR097977.fastq:TTCTTACCATCCTGAAGTTTTTTCATCTTCCCTGAT
> > --
> > SRR098026.fastq-@SRR098026.158 HWUSI-EAS1599_1:2:1:1:1505 length=35
> > SRR098026.fastq:GNNNNNNNNCAAAGTTGATCNNNNNNNNNTGTGCG
> >     ```
> > 
> {: .solution}
{: .challenge}

## Redirecting output

`grep` allowed us to identify sequences in our FASTQ files that match a particular pattern. 
All of these sequences were printed to our terminal screen, but in order to work with these 
sequences and perform other operations on them, we will need to capture that output in some
way. 

We can do this with something called "redirection". The idea is that
we are taking what would ordinarily be printed to the terminal screen and redirecting it to another location. 
In our case, we want to print this information to a file so that we can look at it later and 
use other commands to analyze this data.

The command for redirecting output to a file is `>`.

Let's try out this command and copy all the records (including all four lines of each record) 
in our FASTQ files that contain 
'NNNNNNNNNN' to another file called `bad_reads.txt`.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
~~~
{: .bash}

> ## File extensions
> 
> You might be confused about why we're naming our output file with a `.txt` extension. After all,
> it will be holding FASTQ formatted data that we're extracting from our FASTQ files. Won't it 
> also be a FASTQ file? The answer is, yes - it will be a FASTQ file and it would make sense to 
> name it with a `.fastq` extension. However, using a `.fastq` extension will lead us to problems
> when we move to using wildcards later in this episode. We'll point out where this becomes
> important. For now, it's good that you're thinking about file extensions! 
> 
{: .callout}


The prompt should sit there a little bit, and then it should look like nothing
happened. But type `ls`. You should see a new file called `bad_reads.txt`. 

We can check the number of lines in our new file using a command called `wc`. 
`wc` stands for **word count**. This command counts the number of words, lines, and characters
in a file. 

~~~
$ wc bad_reads.txt
~~~
{: .bash}

~~~
  537  1073 23217 bad_reads.txt
~~~
{: .output}

This will tell us the number of lines, words and characters in the file. If we
want only the number of lines, we can use the `-l` flag for `lines`.

~~~
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
537 bad_reads.txt
~~~
{: .output}

Because we asked `grep` for all four lines of each FASTQ record, we need to divide the output by
four to get the number of sequences that match our search pattern.

> ## Exercise
>
> How many sequences in `SRR098026.fastq` contain at least 3 consecutive Ns?
>
>> ## Solution
>>  
>>
>> ~~~
>> $ grep NNN SRR098026.fastq > bad_reads.txt
>> $ wc -l bad_reads.txt
>> ~~~
>> {: .bash}
>> 
>> ~~~
>> 249
>> ~~~
>> {: .output}
>>
> {: .solution}
{: .challenge}

We might want to search multiple FASTQ files for sequences that match our search pattern.
However, we need to be careful, because each time we use the `>` command to redirect output
to a file, the new output will replace the output that was already present in the file. 
This is called "overwriting" and, just like you don't want to overwrite your video recording
of your kid's first birthday party, you also want to avoid overwriting your data files.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
537 bad_reads.txt
~~~
{: .output}

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR097977.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
0 bad_reads.txt
~~~
{: .output}

Here, the output of our second  call to `wc` shows that we no longer have any lines in our `bad_reads.txt` file. This is 
because the second file we searched (`SRR097977.fastq`) does not contain any lines that match our
search sequence. So our file was overwritten and is now empty.

We can avoid overwriting our files by using the command `>>`. `>>` is known as the "append redirect" and will 
append new output to the end of a file, rather than overwriting it.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
537 bad_reads.txt
~~~
{: .output}

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR097977.fastq >> bad_reads.txt
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
537 bad_reads.txt
~~~
{: .output}

The output of our second call to `wc` shows that we have not overwritten our original data. 

We can also do this with a single line of code by using a wildcard: 

~~~
$ grep -B1 -A2 NNNNNNNNNN *.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~
{: .bash}

~~~
537 bad_reads.txt
~~~
{: .output}

> ## File extensions - part 2
> 
> This is where we would have trouble if we were naming our output file with a `.fastq` extension. 
> If we already had a file called `bad_reads.fastq` (from our previous `grep` practice) 
> and then ran the command above using a `.fastq` extension instead of a `.txt` extension, `grep`
> would give us a warning. 
> 
> ~~~
> grep -B1 -A2 NNNNNNNNNN *.fastq > bad_reads.fastq
> ~~~
> {: .bash}
> 
> ~~~
> grep: input file ‘bad_reads.fastq’ is also the output
> ~~~
> {: .output}
> 
> `grep` is letting you know that the output file `bad_reads.fastq` is also included in your
> `grep` call because it matches the `*.fastq` pattern. Be careful with this as it can lead to
> some unintended results.
> 
{: .callout}

Since we might have multiple different criteria we want to search for, 
creating a new output file each time has the potential to clutter up our workspace. We also
thus far haven't been interested in the actual contents of those files, only in the number of 
reads that we've found. We created the files to store the reads and then counted the lines in 
the file to see how many reads matched our criteria. There's a way to do this, however, that
doesn't require us to create these intermediate files - the pipe command (`|`).

This is probably not a key on
your keyboard you use very much, so let's all take a minute to find that key. 
What `|` does is take the output that is
scrolling by on the terminal and uses that output as input to another command. 
When our output was scrolling by, we might have wished we could slow it down and
look at it, like we can with `less`. Well it turns out that we can! We can redirect our output
from our `grep` call through the `less` command.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | less
~~~
{: .bash}

We can now see the output from our `grep` call within the `less` interface. We can use the up and down arrows 
to scroll through the output and use `q` to exit `less`.

If we don't want to create a file before counting lines of output from our `grep` search, we could directly pipe
the output of the grep search to the command `wc -l`. This can be helpful for investigating your output if you are not sure
you would like to save it to a file. 

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | wc -l 
~~~
{: .bash}

Redirecting output is often not intuitive, and can take some time to get used to. Once you're 
comfortable with redirection, however, you'll be able to combine any number of commands to
do all sorts of exciting things with your data!

None of the command line programs we've been learning
do anything all that impressive on their own, but when you start chaining
them together, you can do some really powerful things very
efficiently. 

