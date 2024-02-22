# Notifications

Event Notification in OpCon is a process in which Machine, Schedule, and Job status change events generate (i.e., trigger) notifications. For critical notifications, administrators can also apply escalation rules to provide higher visibility.

The basic types of notifications include Windows Event Log, Email (SMTP), Network Message, SNMP Trap, Unisys Single Point of Operations (SPO) AL and CO Reports, Text Messages (SMS), OpCon Events, and Command. With OpCon, notifications can be created in several ways.

## Components

The Event Notification system in OpCon consists of the following components:

- **SMA Notify Handler**: Reads the OpCon database and writes the message according to notification type. For additional information, refer to [SMA Notify Handler](../server-programs/notify-handler.md).
- **Notification Manager**: Used to create groups and define notifications based on Machine, Schedule, and Job status change events in OpCon. For more information, including how to create Notification Groups and Triggers, refer to [Using Notification Manager](../Files/UI/Enterprise-Manager/Using-Notification-Manager.md).
  - **Notification Groups**: User-defined groups of either machines, schedules, or jobs. The items in these groups should be selected based on which common triggers and notifications should execute for that group of items.
  - **Notification Triggers**: When Machine, Schedule, and Job status change events occur; database triggers execute based on conditions defined in Notification Manager. Notification details that are defined in Notification Manager are sent to the SMA Notify Handler by way of the database.
- **Escalation Manager**: Used to configure notification escalation groups and rules. For more information, refer to [Using Escalation Manager](../Files/UI/Enterprise-Manager/Using-Escalation-Manager.md).
- **Escalation Acknowledgment**: Used to view escalations and to acknowledge them. For more information, refer to [Using Escalation Acknowledgment](../Files/UI/Enterprise-Manager/Using-Escalation-Acknowlegement.md).

## Escalation

An escalation in OpCon is a notification that has been marked for recurring notification, with an escalation rule, to one or more groups of people (defined within escalation groups). The escalation must be acknowledged to stop the escalation process. When someone acknowledges an escalation, this indicates to OpCon that the user read the notification. From there, it is up to the customer to take any required action.

:::tip Example
There are a group of critical jobs that require immediate attention if any of them fail. To ensure the proper action takes place, the OpCon administrator sets up the following:

- In Escalation Manager, the administrator:
  - Defines three escalation groups to represent **First**, **Second**, and **Third** level contacts.
  - Defines an escalation rule named **Three Level** where:
    - The delay between all repeat notifications is set to 5 minutes.
    - The **First** group is given 3 chances to acknowledge the escalation. If the escalation is not acknowledge in time, then the **Second** group gets notified.
    - The **Second** group is also given 3 chances to acknowledge the escalation. If the escalation is not acknowledge in time, then the **Third** group gets notified.
    - The **Third** group is given 5 chances to acknowledge the escalation. If the escalation is not acknowledge in time, the escalation process will be exhausted.
- In Notification Manager, the administrator:
  - Defines a Job Group with the critical jobs included.
  - Defines a trigger on the Job Group to detect a "Job Failed" event.
  - Defines an Email including all the details explaining the action to take when any of the jobs fail.
  - Applies the **Three Level** escalation rule for the trigger.

As as result of this configuration:

- When any of the jobs in the Job Group fail, notifications will go out to the **First** level escalation group right away.
- Anyone who can see the escalation in the Escalation Acknowledgment view in the EM can acknowledge the notification.
- The person who stops the escalation process should then immediately take care of the failed job.
- If the escalation is not acknowledge in time, the escalation process continues to the **Second** and **Third** groups.

:::

## Notification Triggers

Notification triggers are responsible for writing the notification information to the OpCon database upon the status change of one or more items within a group.

- Machine and Job triggers can be placed on Machine groups.
- Schedule and Job triggers can be placed on Schedule groups.
- Job triggers can be placed on Job groups.

:::tip Example
In Notification Manager, a trigger for Job Failed is set on a Job Group of jobs containing a job named "Backup." The job "Backup" is configured to run on each machine in a group.

When the schedule builds, the single job from Job Master will become several jobs that display in any Operation view with the following names:

- Backup_WinMach1
- Backup_WinMach2
- Backup_WinMach3

Because "Backup" was selected in the Job Group, if any one of the jobs fail, the notification will be sent and the exact name of the job can be sent with the notification so customers know which instance of the job failed.
:::

### Machine Triggers

|Status|Definition|
|--- |--- |
|Machine Marked Down|Machine has been marked down by a user.|
|Machine Marked Limited|Job Starts have been disabled on the machine.|
|Machine Marked Up|Machine has been marked up by a user.|
|Machine Status Changed|Machine has been marked up, limited (i.e., disabled job starts), or down by a user.|
|Network Down|SMANetCom has lost communication with the LSAM.|
|Network Up|SMANetCom has re-established communication with the LSAM.|
|Network Status Changed|Communication status has changed since the last time the SMANetCom polled the LSAM.|

### Schedule Triggers

A schedule qualifies for a schedule event trigger if it is a schedule in the Daily tables that contains jobs. The trigger does not fire just because the name of the schedule appears in a schedule name that has been constructed as part of a subschedule. For a subschedule, the schedule name at the end of the concatenated schedule name is the one that will qualify for the trigger if it is in the Notification Manager Group.

:::tip Example
In Notification Manager, a trigger for a schedule placed on Hold is set on a Notification Manager Group of schedules containing a schedule named "ImportantProcess". This schedule is configured as a subschedule on other schedules, and it also has Container jobs on it to call other schedules.

When the schedules build for the day, the following schedules contain the schedule name:

1. ImportantProcess
2. ImportantProcess_Container1\[OtherSchedule]
3. ProcessFiles_Container1\[ImportantProcess]
4. ProcessFiles_Container1\[ImportantProcess]_Container1\[OtherSchedule]

Because ImportantProcess is in the Notification Manager Group for the trigger, if schedule 1 or 3 is placed On Hold, the trigger will fire. It does not fire for schedules 2 or 4 because "OtherSchedule" is the schedule that contains jobs. The trigger would fire for those schedules only if "OtherSchedule" was in the Notification Manager Group.
:::

|Status|Definition|
|--- |--- |
|Schedule Start|The time qualification has been met and the schedule has started. Any subschedules in a status of Parent Hold that were Started by the parent being started will not cause this trigger to fire. Only notification for the parent schedule status change will occur.|
|Schedule Complete|All jobs on the schedule have finished and the schedule is now complete.|
|Schedule Placed On Hold|An OpCon event or a user has placed the schedule on hold.  Any subschedules in a status of Parent Hold will not cause this trigger to fire. Only notification for the parent schedule status change will occur.|
|Schedule Released From Hold|An OpCon event or a user has released the schedule from hold.  Any subschedules in a status of Parent Hold that were released by the parent being released will not cause this trigger to fire. Only notification for the parent schedule status change will occur.|

### Job Triggers

A job qualifies for a job event trigger if the name of the job in the Daily was derived from the selected job in the Master.

:::tip Example
In Notification Manager, a trigger for Job Failed is set on a Notification Manager Group of jobs containing a job named "Backup." The job "Backup" is configured to run on each machine in a group.

When the schedule builds, the single job from Master will become several jobs in the Daily with the names:

- Backup_WinMach1
- Backup_WinMach2
- Backup_WinMach3

Because "Backup" was selected in the Notification Manager Group, if any one of the jobs in the Daily fail, the notification will be sent and the exact name of the job can be sent with the notification so customers know which instance of the job failed.
:::

|Status|Definition|
|--- |--- |
|Job Wait Machine|The job has entered a Wait Machine status. If a machine is placed into a Limited state (i.e., job starts disabled), all jobs that have qualified to run on that machine go into a Wait Machine state. If notifications are set up based on a Job Wait Machine trigger, unwanted notifications may occur. |
|Job Late to Start|The Job Late to Start trigger is fired when SAM sets a job status to Late to Start.|
|Job Late to Finish|The Job Late to Finish trigger is fired when SAM sets a job status to Running; Late to Finish.|
|Job Submitted|The SAM has sent the information (via SMANetCom) to the LSAM for a job to be started.|
|Job Still Attempting to Start|The Job Still Attempting to Start trigger is fired when SAM checks on the job's status and the job is still in a Start Attempted status. To change the frequency at which this occurs, change the "Minutes between checking running jobs" parameter in the "Time Settings" category of the Advanced Configuration for Machine <machine name\> screen. Refer to Time Settings.|
|Job Initialization Error|The job was unable to start for reasons on the server or agent including when the job was submitted to the operating system there was a failure (e.g., the Batch User ID was invalid) or the OpCon Server was unable to resolve tokens for the command line or any other precondition of the job.|
|Job Prerun Failed|The job used as a Prerun has finished with an exit condition that did not meet the defined Failure Criteria.|
|Job Missed Start Time|A job has not been submitted for starting because the latest start time has passed.|
|Job Running|The Job Running trigger fires in three possible scenarios:A job goes from Submitted to Running.A job goes from Submitted to Finished OK. A job only goes straight from Submitted to Finished OK for a very short running job that never returns a Running status.A job goes from Submitted to Failed. A job only goes straight from Submitted to Failed for a very short running job that never returns a Running status. This trigger does not fire for a job that goes straight to a prerun or initialization failure.|
|Job to be Killed|A job kill command has been issued through a graphical interface or through a $JOB:KILL event. The trigger fires when the SAM sends the kill message to the LSAM. The trigger does not fire if the job was killed manually on the LSAM machine.|
|Job Exceeded Max Runtime|The job has been running longer than the user-defined maximum runtime.|
|Job Finished OK|The job has completed with a status of Finished OK.|
|Job Failed|The job has completed with a status of Failed.|
|Job Marked Finished OK|The job was marked Finished OK by an OpCon event or a user.|
|Job Marked Failed|The job was marked Failed by an OpCon event or a user.|
|Job Cancelled|A job was cancelled by an OpCon event or a user.|
|Job Restarted|A job was placed back in a Qualifying state by an OpCon event or a user.|
|Job Skipped|A job was placed in a Skipped status by an OpCon event or a user.|
|Job Under Review|A job was placed in an Under Review status by an OpCon event or a user.|
|Job Fixed|A job was placed in a Fixed status by an OpCon event or a user.|