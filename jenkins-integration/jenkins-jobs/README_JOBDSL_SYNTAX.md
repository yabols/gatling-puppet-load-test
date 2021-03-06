## gatling-puppet-load-test `JobDSL.groovy` syntax

In each scenario directory where you have created a `Jenkinsfile` to define a
gatling-puppet-load-test perf testing job, you may optionally also create a file
named `JobDSL.groovy`.  If such a file is found, the seed job will use it to
configure additional metadata about the job.

If this doesn't make sense yet, make sure you've read the [main README.md](./README.md)
about defining perf testing jobs, and the [Jenkinsfile README.md](./README_JENKINSFILE_SYNTAX.md)
about what the `Jenkinsfile`s look like.

Basically, the `Jenkinsfile` is used to define what happens once a job is
launched, and the `JobDSL.groovy` file (optionally) configures metadata about the
job before it is launched.  Examples might be:

* modifying the number of runs to retain in the build history for a job
* scheduling the job to automatically run at a certain time interval

The `JobDSL.groovy` file should just be a plain-old groovy script (no need to
define any classes or functions, though you should be able to define functions
for your own use if you like).  It uses the [Jenkins JobDSL plugin](https://github.com/jenkinsci/job-dsl-plugin/wiki).
There is some [pretty solid API documentation for this DSL](https://jenkinsci.github.io/job-dsl-plugin/)
available if you're not sure how to modify some type of job configuration that
you are interested in.  (It's worth calling out again that the JobDSL DSL/API
is completely distinct from the Jenkins Pipeline DSL that is used in the
`Jenkinsfile`.)

When the script is run, it will be executed with
a binding context that makes two variables available in its local scope:

* `out`: this is the [JobDSL logging stream](https://github.com/jenkinsci/job-dsl-plugin/wiki/Job-DSL-Commands#logging).
  Basically, you can call `println` on this object to have log messages show up
  in the Jenkins console output when the seed job (`refresh-gplt-jobs`) is run.
* `job`: this variable will contain a reference to
  [the actual job definition](https://github.com/jenkinsci/job-dsl-plugin/wiki/Job-DSL-Commands#job)
  that the seed job is building up.  To make changes to the job, you call `job.with { ... }`, where the code inside
  of those braces will be calls to the JobDSL to modify the job.  Here's an example of how you could change the
  maximum number of builds stored in the job history to 10:

```groovy
job.with {
    logRotator {
        numToKeep(10)
    }
}
```

* `helper`: this variable has an instance of the `DSLHelper` class that is defined in the `job_bootstrap.groovy` file.
  The purpose of it is to encapsulate some common, re-usable job configuration logic that may be useful in many of our
  jobs, but would be a little unwieldy to duplicate in every `JobDSL.groovy` that might want to take advantage of it.
  At the time of this writing, the only method is `overrideParameterDefault(job, param_name, new_default_value)`, which
  allows you to override the default value for a parameter such as `SUT_HOST`.

* `serverConfig`: this is a map, containing some basic configuration information about the server that the job is
  being created on.  The data for this is loaded from `./server_config.json`.  The keys that it will contain are:

  * "hostname": the hostname of the server.
  * "environment": an identifier that can be used to distinguish between different classes of servers; most common
    values will be either "development" or "production".  You can use this as a condition in your JobDSL script,
    e.g. to set up a time-based trigger for a job on production without doing so on dev servers.

Theoretically you should be able to use any feature of the JobDSL within these scripts; refer to the
[JobDSL API](https://jenkinsci.github.io/job-dsl-plugin/) for more info.
