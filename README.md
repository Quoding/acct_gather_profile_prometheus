# acct_gather_profile_prometheus
## Description
A profiling plugin for Slurm which sends the data to the Prometheus Pushgateway

## How to activate it

Once you got the acct_gather_profile_prometheus.so file. Put it into Slurm's lib64 folder
where all the other accounting plugins also are (on Magic_Castle, this is `/opt/slurm/lib64/slurm`).
Once this is done, you can edit Slurm's configuration file (`/etc/slurm/slurm.conf`) and add this entry:

`AcctGatherProfileType=acct_gather_profile/prometheus`

and in `/etc/slurm/acct_gather_profile.conf` (if it's not already there, feel free to create it), add the following:

`ProfilePrometheusHost=URL_OF_YOUR_PUSHGATEWAY:PORT_OF_PUSHGATEWAY`

For example, if you want one pushgateway per compute node, you can specify `ProfilePrometheusHost=http://localhost:9091`.

You can also specify `ProfilePrometheusDefault=<all|none|[energy[,|task[,|filesystem[,|network]]]]>`
to force the profiling options, however, so far only task profiling was tested.

At this point, all you should have to do is add `#SBATCH --profile=<all|none|[energy[,|task[,|filesystem[,|network]]]]>` to your submit script
and everything should be good to go!

## How it works

![alt text](https://docs.google.com/drawings/d/e/2PACX-1vTfNbhKJjLL1YRm1IJ0J_Ga5k9mFkeEKUUMnppC3NiA6SsKUVtQ7HKFR-9aosTzhuKdPQmt9yUYP9wr/pub?w=1315&h=704 "Prometheus Plugin Diagram")

Due to Prometheus's pull architecture, a Slurm job, because of its ephemereal
nature, has to push data somewhere. In comes the Prometheus Pushgateway which
enables just that. According to Slurm's documentation, 

`int acct_gather_profile_p_add_sample_data(uint32_t type, void* data);`

> Put data at the Node Samples level. Typically called from something called at either job_acct_gather interval or acct_gather_energy interval.
All samples in the same group will eventually be consolidated in one time series.

In the plugin, this function is where the data is put into a Prometheus's
scrapable string. It is also where the plugin sends it data to Prometheus's
Pushgateway via curl (libcurl) inside the **_send_data()** function. Once the Slurm job ends,
it automatically calls

`int acct_gather_profile_p_task_end(stepd_step_rec_t* job, pid_t taskpid)`

> Called once per task from slurmstepd.
Provides an opportunity to put final data for a task. 

This function calls **_delete_data()**, which sends a DELETE request via curl
(libcurl) to remove the data for a specified jobid and node name.

In any case, the data pushed to the Pushgateway are scraped by Prometheus at a
given interval. The DELETE request is sent in order to delete the data associated
with a finished job (otherwise, the Pushgateway would keep this data indefinitely
and Prometheus would scrape this data over and over, resulting in a flat line on a 
graph, for example.)