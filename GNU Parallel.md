# Check number of cores
```
grep -c ^processor /proc/cpuinfo
nproc --all
```

# Run a text process script in parallel
* Save the commands into a runlist 
```
...
processtext file1
processtext file2
processtext file3
...
```
* Run the runlist with parallel
```
parallel < runlist
```
**Note:** Do not let the commands output to the same file.

# References:
* [GNU Parallel](https://www.gnu.org/software/parallel/)
