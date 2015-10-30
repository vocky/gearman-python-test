Gearman python test
===================
Use gearman to build clusters.
Quick-start
-----------

1.Install

.. code-block:: console

    $ sudo apt-get install gearman-job-server
    $ sudo pip install gearman

2.Start Jobs

.. code-block:: coonsole

    $ gearmand -l gearman.log -L 172.26.183.16 -p 4735 &
    $ gearmand -l gearman.log -L 172.26.183.15 -p 4735 &

3.Start workers in clusters

.. code-block:: python

    import gearman
    gm_worker = gearman.GearmanWorker(['172.26.183.16:4735', '172.26.183.15:4735'])
    def task_listener_reverse(gearman_worker, gearman_job):
    print 'Reversing string: ' + gearman_job.data
    return gearman_job.data[::-1]

    # gm_worker.set_client_id is optional
    # gm_worker.set_client_id('python-worker')
    gm_worker.register_task('reverse', task_listener_reverse)
    # Enter our work loop and call gm_worker.after_poll() after each time we timeout/see socket activity
    gm_worker.work()

4.Start client

.. code-block:: python

    import gearman

    # setup client, connect to Gearman HQ

    def check_request_status(job_request):
        if job_request.complete:
            print "Job %s finished!  Result: %s - %s" % (job_request.job.unique, job_request.state, job_request.result)
        elif job_request.timed_out:
            print "Job %s timed out!" % job_request.unique
        elif job_request.state == JOB_UNKNOWN:
            print "Job %s connection failed!" % job_request.unique

    gm_client = gearman.GearmanClient(['172.26.183.16:4735', '172.26.183.15:4735'])

    word = 'Hello World!'
    completed_job_request = gm_client.submit_job("reverse", word)
    check_request_status(completed_job_request)

