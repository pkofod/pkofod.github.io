---
layout: post
title: Running many similar jobs using SLURM and Matlab
author: Patrick Kofod Mogensen
excerpt: Running many similar jobs on a server with the SLURM queuing software is easy to do - even if you have many parameters to vary!
---

# SLURM

SLURM stands for Simple Linux Utility for Resource Management. It essentially allows the users to submit jobs to a queue, and when there is sufficient ressources available (and it is your turn), the job will start. Additionally, SLURM can send you e-mails with progress, save output of your program to a file, write to a log file, and so on. To submit a job, you make a bash-file, and pass it to `sbatch`.

```shell
sbatch job.sh
```

Where `job.sh` contains the information to be passed to SLURM. The options for SLURM and the code to be run.

```shell
#!/bin/bash
#SBATCH --job-name=MyRun
#SBATCH --mail-type=ALL
#SBATCH --mail-user=my@mail.com

ls > my_folder.list
```

After runing `sbatch`, we would have a file called `my_folder.list` with the contents of the folder we were in. Great stuff. Now let's run a Matlab script (or R, python, Julia, whatever). If we were simply in a terminal, we could do
```shell
matlab -nodesktop -r my_script.m
# or
matlab -nodesktop < my_script.m
```
They are not quite identical, but let's leave the details be. You should note that the first does not quit matlab after the script is run, but the second does. If we only want to run one script this may be fine, but if we want to run many similar scripts, this is not the best way to do it. We would have to have as many scripts as jobs, and that is a file mess. We could generate them with a script, but we will avoid that.

# Here documents
First, we will have to look at so-called here documents. To understand them, let's try with a simple example.
```shell
wall << EndOfMessage
Hello Friends!
EndOfMessage
```
`wall` is a linux program that writes a message to all logged on users. What the above example does, is the following: `<< EndOfMessage` tells your computer, that we are going to pass some input to the program `wall`, and it is going to be everything that fellows until we meet the expression `EndOfMessage` if it starts on a new line with no white space around it. The name was chosen by me, and could easily have been `EOM` for short, you just don't want to choose a keyword, which might be used in the particular context - for example `end`. Similarly, we could pass a script to Matlab. Simply write the following.

```shell
matlab -nodesktop << EOF
A = [1 3 9];
B = [2 5 0];
C = A + B;
output = mean(C) == 42
EOF
```
And we will learn that the `mean` of `C` is sadly not `42`. However, we also learnt how to use the super useful here document feature. Let us try a different example. We will create a 3x3 matrix, and use an index to choose which column to calculate the mean of.

```shell
matlab -nodesktop << EOF
A = [1 3 9; 9 3 1; 2 2 116];
matrix_index = 3;
output = mean(A(:, matrix_index)) == 42
EOF
```
And we see that it is indeed true! How convenient.
# SLURM arrays
So, I promised something about running similar jobs. This is where the `--array` option to `sbatch` comes into play. Let us make a SLURM job file as above. This time, it runs our little `mean`-script instead of running `ls`. The following should be saved in a file, for example `myjobs.sh`, and is run by typing `sbatch myjobs.sh`.

```shell
#!/bin/bash
#SBATCH --job-name=MyRun
#SBATCH --output=MyOutput-%j.out
#SBATCH --mail-type=ALL
#SBATCH --mail-user=my@mail.com
#SBATCH --array 1-3

matlab << EOF
A = [1 3 9; 9 3 1; 2 2 116];
matrix_index = $SLURM_ARRAY_TASK_ID
output = mean(A(:, matrix_index)) == 42
EOF
```

This time I've added some new things: `--output` and `array` as options to `sbatch`, `%j%`, and `$SLURM_ARRAY_TASK_ID`. I've also omitted the `-nodesktop` option, as the GUI is probably not even installed on the server where you run `sbatch`. The `--output` option adds the possibility for `sbatch` to save the output (of Matlab in this instance) to a file. The `%j%` refers to the job ID given by SLURM to your job. The `--array` option will run our script three times. Each time it runs, it will substitute a value for `$SLURM_ARRAY_TASK_ID`. It will start with 1, then 2, and lastly 3. However, it is not sure that SLURM will actually run them in that order. Checking the `.out` files, you should be able to see, that the job with `matrix_index == 3` is the only job which returns `output` as `true` (or `1`).