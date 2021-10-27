## Jobs
Kubernetes includes support for creation of pods that terminates after process running inside finishes successfully and not restarted again.

In the event of a node failure, the pods on that node that are managed by a Job will be rescheduled to other nodes.

In the event of a failure of the process itself (when the process returns an error exit code), the Job can be configured to either restart the container or not.

* Run multiple pod instances in a Job
    * Run job pods sequentially
    * Run job pods in parallel
    * Scale a Job
* Limit the time allowed for a Job pod to complete    

## CronJobs
Job resources run their pods immediately when you create the Job resource. But many batch jobs need to be run at a specific time in the future or repeatedly in the specified interval. In Linux- and UNIX-like operating systems, these jobs are better known as cron jobs. Kubernetes supports them, too.

A cron job in Kubernetes is configured by creating a CronJob resource. At the configured time, Kubernetes will create a Job resource according to the Job template configured in the CronJob object. When the Job resource is created, one or more pod replicas will be created and started according to the Job’s pod template. 
