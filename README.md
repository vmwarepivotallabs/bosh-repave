# Bosh Repave

This is a sample of a Concourse pipeline that will perform the repave of VMs of selected bosh deployments in a PCF deployment (existence of Ops Manager VM required) on a pre-defined schedule.

The pipeline also allows for the selection of the VMs to be repaved within each deployment, either ALL or only ones of Non-Singleton instances.

The pipeline scripts use `bosh recreate` as the mechanism to perform the repave of VMs.

The scripts iterate through the list of selected deployments provided as pipeline parameters (e.g. `cf, apm, ...`) and then issue the `bosh recreate` command for the targeted instances.


### Notes

- The purpose of the pipeline is to provide a mechanism to selectively repave VMs of selected deployments on a regular basis, for organizations that require such procedure for any reason such as internal security regulations.  

- Using and running this pipeline should be done with proper caution and planning, as the recreation of VMs that implement singleton jobs may cause outages to the platform. The pipeline provides a flag to skip VMs of singleton jobs.   

- This pipeline should be scheduled to run in a time-window that does not coincide with actions that may affect the platform availability or performance, such as backup, updates or upgrades. A scheduler (time resource) is provided by default in the pipeline implementation.  

- In addition to the scheduler, a "deployments lock" mechanism is implemented with the pipeline. Such mechanism should also be used in other pipelines such as the ones for backup and upgrades, so these pipelines don't run into each other's execution.  


### How to use the pipeline

1. Clone this project locally
1. Make a copy of `ci/secrets.sample.yml`:  
   `cp ci/secrets.sample.yml ci/secrets.yml`
1. Edit the copy of the secrets file:  
   `opsman-url`: Ops Manager URL. e.g.  https://pcf.example.com  
   `opsman-username`: Ops Manager admin user. e.g. admin  
   `opsman-password`: Ops Manager user password  
   `skip-ssl-validation`: skip ssl validation for Ops Manager login e.g. `true` or `false`  
   `deployments`: comma-separated list of bosh deployments to repave. It has to contain the prefix of PCF deployed releases (from the output of `bosh deployments`, remove the `-XXXXXXX...` numeric suffix from the deployment name generated by PCF), e.g. `cf,apm`  
   `repave-singleton-jobs`: flag to control which VM instances will be repaved. e.g. `true` repave only jobs with more than one instance, `false` repave all VMs of all jobs.  
   `perform-dry-run-only`: for testing the repave action with a _dry-run_ of _bosh recreate_ without actually recreating any VMs. e.g. `true` performs dry-run, `false` executes repave for real   
   `scheduler-time-window-start`: initial time of the scheduler job execution time window, e.g. `1:00 AM`  
   `scheduler-time-window-stop`: end time of the scheduler job execution time window, e.g. `2:00 AM`
   `scheduler-time-location`: time zone for the scheduler. e.g. `America/Phoenix`. See [docs](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for accepted values.  
   `lock-git-repo-uri`: The git repo URL for the pool resource used as a deployment lock. See [docs](https://github.com/concourse/pool-resource) for more information on how to bootstrap a pool resource repository.  
   `lock-git-repo-branch`: git branch for the lock repository. e.g. `master`  
   `lock-pool-name`: the pool name for the lock  
   `lock-git-private-key`: private key for the pool resource repository  

1. Create the pipeline with the `fly` command  
   e.g. `fly -t <target> sp -p repave -c ci/pipeline.yml -l ci/secrets.yml`  

1. Unpause the pipeline to either run it manually or when the scheduler generates an event within the specified time-window.  
