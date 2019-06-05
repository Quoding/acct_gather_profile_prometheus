# acct_gather_profile_prometheus
## Description
A profiling plugin for Slurm which sends the data to the Prometheus Pushgateway

## How it works

![alt text](https://docs.google.com/drawings/d/e/2PACX-1vTfNbhKJjLL1YRm1IJ0J_Ga5k9mFkeEKUUMnppC3NiA6SsKUVtQ7HKFR-9aosTzhuKdPQmt9yUYP9wr/pub?w=1315&h=704 "Prometheus Plugin Diagram")

Due to Prometheus' pull architecture, a Slurm job, because of its ephemereal
nature, has to push data somewhere. In comes the Prometheus Pushgateway which
enables just that. According to Slurm's documentation, 
> int acct_gather_profile_p_add_sample_data(uint32_t type, void* data); 

> Put data at the Node Samples level. Typically called from something called at either job_acct_gather interval or acct_gather_energy interval.
All samples in the same group will eventually be consolidated in one time series.

In the plugin, this function is where the data is put into a Prometheus'
scrapable string. It is also where the plugin sends it data to Prometheus'
Pushgateway via curl (libcurl) inside the **_send_data()** function. Once the job ends,
it automatically calls

> int acct_gather_profile_p_task_end(stepd_step_rec_t* job, pid_t taskpid)

> Called once per task from slurmstepd.
Provides an opportunity to put final data for a task. 

This function calls **_delete_data()**, which sends a DELETE request via curl
(libcurl) to remove the data for a specified jobid and node name.

In any case, the data pushed to the Pushgateway are scraped by Prometheus at a
given interval. The DELETE request is sent in order to delete the data associated
with a finished job (otherwise, the Pushgateway would keep this data indefinitely
and Prometheus would scrape this data over and over, resulting in a flat line on a 
graph, for example.)