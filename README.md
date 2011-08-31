Logging To Logentries from Google AppEngine using Python
========================================================

Logentries currently provides 3 methods of logging to our servers.

In-Process Logging, Push-Queue Logging and Pull-Queue Logging.

--------------------------------------------------------------

In-Process Logging may have a performance impact and is advised for development only. 
It is limited by App Engine's urlfetch api quotas. (1 api call per log)
 
Push-Queue Logging has minimal impact on performance and is advised for low log volume 
production systems, however it too is limited by App Engine's push-queue quotas. 
This method is suitable for a system logging no more than 100,000 logs a day using a 
free account or 20,000,000 logs a day using a paid account.

Pull-Queue Logging has no impact on performance and is advised for high log volume 
production systems. It is not limited by App Engine quotas and is suited to systems
that expect high logging volumes.

-----------------------------------------------------------------------------------

Before implementing your chosen option, you need to create an account on Logentries. Once you have done this,
you will need to download the getKey.py script on github which is necessary for getting your user-key.
This user-key is required for each of the steps listed below and is referred to below simply as key.

Once you have downloaded the script run it as follows `python getKey.py --key`.  It will prompt you for
your login credentials and then print out your user-key to be used below.

You can create your host(s) and log(s) either using the Logentries UI or with the script mentioned above.

Running the command `python get.py --register` will set up a default host called AppEngine and a default log
called AppEngine.log or you can specify your own names for the host or log using the parameters:
`-h hostname` and `-l logname`

In-Process Logging
------------------

To Enable In-Process logging in your app, you must add logentries.py to your directoy, it is available on here
at  
	https://github.com/downloads/logentries/le_appengine_py/In-Process.zip

Now if you don't already have an appengine_config.py simple create one in your app's directory.

Add the following lines to this config file:

	import logentries

	logentries.init('KEY', 'LOCATION')

You will notice the two parameters above called key and location.

  - Key is your unique password to the site and must be kept secret. As mentioned earlier this key is
  obtained by running `python getKey.py --key`
  
  - Location is the name of your host on logentries followed by the name of the log, e.g 'hostname/logname'
  Running `python getKey.py --register` will set up the following default   `AppEngine/AppEngine.log` 

Once this is done properly, simple import logging in the files you wish to log from and use the python 
logging module as normal for it to log to Logentries also.

For example:  

            logging.info("informational message")
            logging.warn("warning message")
            logging.crit("critical message")

Push-Queue Logging
------------------

To Enable Push-Queue logging in your app, you must add 2 files to your app's directory,  
le.py and logentries.py available at  

          https://github.com/downloads/logentries/le_appengine_py/Push-Queue.zip

Next you need to import both logging and logentries in the file that you declare
your webapp.WSGIApplication(...) handlers.
like so:

         import logging, logentries

and then add the following handler:

         ('/logentriesworker', logentries.LogentriesWorker)
         
This is the worker that will be used to transmit the logs to Logentries.

If you don't already have an appengine_config.py file in your app, simple create a new file by that name.

In this file add the following lines:

         import logentries
         
         logentries.init('KEY', 'LOCATION')

You will notice the two parameters above called key and location.

  - Key is your unique password to the site and must be kept secret. As mentioned earlier this key is
  obtained by running `python getKey.py --key`
  
  - Location is the name of your host on logentries followed by the name of the log, e.g 'hostname/logname'
  Running `python getKey.py --register` will set up the following default   `AppEngine/AppEngine.log` 
  
Finally add the following to your queue.yaml file or create one if needs be:

         queue:
         - name: logentries
           rate: 5/s
           retry_parameters:
             task_retry_limit: 1
             task_age_limit: 5s
             
These are simply the settings for the pushqueue that our logger will use.

Once this is done properly, simple import logging in the files you wish to log from and use the python 
logging module as normal for it to log to Logentries also.

For example:  

            logging.info("informational message")
            logging.warn("warning message")
            logging.crit("critical message")


Pull-Queue Logging
------------------

To Enable Pull-Queue Logging on your app, you must add 2 files to your app's directory,  
logentriesbackend.py and logentries.py available at  

         https://github.com/downloads/logentries/le_appengine_py/Pull-Queue.zip

If you don't already have an appengine_config.py file in your app, simple create a new file by that name.

In this file add the following lines:

         import logentries
         
         logentries.init('KEY', 'LOCATION')

You will notice the two parameters above called key and location.

  - Key is your unique password to the site and must be kept secret. As mentioned earlier this key is
  obtained by running `python getKey.py --key`
  
  - Location is the name of your host on logentries followed by the name of the log, e.g 'hostname/logname'
  Running `python getKey.py --register` will set up the following default   `AppEngine/AppEngine.log` 

Create a file called backends.yaml with the following contents:

         backends:
         - name: logentriesworker
           class: B1
           instances: 1
           start: logentriesbackend.py


Create a file called queue.yaml with the following contents:

        queue:
        - name: logentries_pull_queue
          mode: pull
          acl:
          - user_email: "put_your_email_here"
  

Once this is done properly, simple import logging in the files you wish to log from and use the python 
logging module as normal for it to log to Logentries also.

For example:  

            logging.info("informational message")
            logging.warn("warning message")
            logging.crit("critical message")
