To run the code and gather darshan logs for it (assuming darshan is installed):

1. Set the environmental variables `LD_PRELOAD` and `DARSHAN_LOGPATH` for Darshan to be activated

2. Compile the simple example using cmake

```
mkdir buid
cd build
cmake ..
make

mpirun -n 4 ./simple temp
```

3. Parse the traces generated by Darshan in `$DARSHAN_LOGPATH/year/month/day/`

```
{darshan-install-path}/bin/darshan-job-summary.pl darshan/simple-io-trace.darshan 
{darshan-install-path}//bin/darshan-parser darshan/simple-io-trace.darshan > darshan/simple-io-trace.darshan.log

```

Example outputs are presented in the `darshan` folder.


4. Execution on Summit

Submit using the batch script provided in the root folder.

```
bsub batch_summit.sh
```
