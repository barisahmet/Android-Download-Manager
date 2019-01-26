[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-Android--Download--Manager-brightgreen.svg?style=flat)](http://android-arsenal.com/details/1/2131)
[![The MIT License](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/majidgolshadi/Android-Download-Manager-Pro/blob/master/LICENSE)
[![](https://jitpack.io/v/barisahmet/Android-Download-Manager.svg)](https://jitpack.io/#barisahmet/Android-Download-Manager)

Android-Download-Manager
========================

Android/Java download manager library help you to download files in parallel mechanism in some chunks.

Overview
========

This library is a download manager android/java library which developers can use in their apps and allow you to download files in parallel mechanism in some chunks and notify developers about tasks status (any download file process is a task). Each download task cross 6 stats in its lifetime.

1. init
2. ready
3. downloading
4. paused
5. download finished
6. end

![applications states](docs/images/states.jpg)

Usage
=====

Step 1. Add the JitPack repository to your build file

```
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
```

Step 2. Add the dependency

```
dependencies {
    compile 'com.github.barisahmet:Android-Download-Manager-Pro:0.0.1'
}
```

### Permission Warning

Normally, library contains WRITE_EXTERNAL_STORAGE permission in its manifest but after Marshmallow(6.0, API23), Android introduced permission requests at runtime.
If you are developing for latest Android versions, you should take care of getting permissions inside your app before calling the library to make a download.

This link http://developer.android.com/training/permissions/requesting.html will help you get started.

TODO: Library will provide an interface to get this permission for you.

Let's get started
=================

One of the important benefits of this lib is that you don't need to initialize object completely before getting any reports.
```java
DownloadManagerPro dm = new DownloadManagerPro(Context);
```
To get report about tasks you can use these methods that will be introduced later on this doc:
```java
public ReportStructure singleDownloadStatus(int token);
public List<ReportStructure> downloadTasksInSameState(int state);
public List<ReportStructure> lastCompletedDownloads();
public boolean unNotifiedChecked();
public boolean delete(int token, boolean deleteTaskFile);
```

>Attention: in this documentation dm stands for DownloadManagerPro object

Initialize DownloadManagerPro
=============================

In order to download with this lib you need to set its basic configurations and give him a listener to poke you about tasks status.
```java
void DownloadManagerPro.init(String saveFilePath, int maxChunk, DownloadManagerListener class)
```

* String **saveFilePath**: folder address that you want to save your completed download task in it.
* int **maxChunk** : number of maximum chunks. any task is divided into some chunks and download them in parallel. it's better not to define more than 16 chunks; but if you do it's set to 16 automatically.
* DownloadManagerListener **listenerClass** in this package an interface created to report developer download tasks status. this interface includes some abstract methods that will be introduced later.

Example:
```java
public class MyActivity extends Activity implements DownloadManagerListener {
    ...
    public void methodName() {
        ...
        // you can only pass this for context but here i want to show clearly
        DownloadManagerPro dm = new DownloadManagerPro(this.getApplicationContext());
        dm.init("downloadManager/", 12, this);
        ...
    }
    ...
}
```

-----------------

There are three ways to define your download task, so you can define it any way you want. for example If you didn't set maximum chunks number or sd card folder address it uses your initialized values. these methods return you a task id that you can call to start or pause that task using this token.
```java
int DownloadManagerPro.addTask(String saveName, String url, int chunk, String sdCardFolderAddress, boolean overwrite, boolean priority)

int DownloadManagerPro.addTask(String saveName, String url, String sdCardFolderAddress, boolean overwrite, boolean priority)

int DownloadManagerPro.addTask(String saveName, String url, boolean overwrite, boolean priority)
```
* String **saveName**: defining te name of desired download file.
* String **url** : Location of desired downlaod file.
* int chunk : Number of chunks which download file has been divided into.
* String sdCardFolder : Location of where user want to save the file.
* boolean **overwrite** : Overwrite if exists another file with the same name. If true, overwrite and replace the file. If false, find new name and save it with new name.
* boolean **priority** : Grant priority to more desired files to be downloaded.

* **return** int **task id**: task token

Example:
```java
int taskToken = dm.addTask("save_name", "http://www.site.com/video/ss.mp4", false, false);
```

----

this method usage is to start a download task. If download task doesn't get started since this task is in downloading state, it throw you an IOException. When download task start to download this lib notify you with OnDownloadStarted interface
```java
void DownloadManagerPro.startDownload(int token) throws IOException
```
* int **token**: It is an assigned token to each new download which is considered as download task id.

Example:
```java
try {
    dm.startDownload(taskToken);

} catch (IOException e) {
    e.printStackTrace();
}
```

----

pause a download tasks that you mention and when that task paused this lib notify you with OnDownloadPaused interface
```java
void DownloadManagerPro.pauseDownload(int token)
```
* int **token**: It is an assigned token to each new download which is considered as download task id.

Example:
```java
dm.pauseDownload(taskToken);
```

----

StartQueueDownload method create a queue sort on what you want and start download queue tasks with downloadTaskPerTime number simultaneously. If download tasks are running in queue and you try to start it again it throws a QueueDownloadInProgressException exception.
```java
void DownloadManagerPro.StartQueueDownload(int downloadTaskPerTime, int priority) throws QueueDownloadInProgressException
```
int downloadTaskPerTime: the number of task that can be downloaded simultaneously
* int priority: Grant priority to more desired files to be downloaded.

 * QueueSort.HighPriority : only high priority
 * QueueSort.LowPriority : only low priority
 * QueueSort.HighToLowPriority : sort queue from high to low priority
 * QueueSort.LowToHighPriority : sort queue from low to high priority
 * QueueSort.earlierFirst : sort queue from earlier to oldest tasks
 * QueueSort.oldestFirst : sort queue from old to earlier tasks


Example:
``` java
try {
    dm.startQueueDownload(3, QueueSort.oldestFirst);

} catch (QueueDownloadInProgressException e) {
    e.printStackTrace();
}
```

-----


this method pauses queue download and if no queue download was started it throws a QueueDownloadNotStartedException exception.
```java
void DownloadManagerPro.pauseQueueDownload()throws QueueDownloadNotStartedException
```

Example:

```java
try {
    dm.pauseQueueDownload();

} catch (QueueDownloadNotStartedException e){
    e.printStackTrace();
}
```

Report
======
In this section we are working with reports since we need to get tasks status and some useful information about those status.

----------

It reports task download information in "ReportStructure" style using a token (download task id) and finally returns the statue of that token.
```java
ReportStruct DownloadManagerPro.SingleDownloadStatus(int token)
```

* int **token**: task token

* return ReportStructure object and it has a method to convert these info to json

* int **id**: task token
* String **name**: file name that will be saved on your sdCard
* int **state**: download state number
* String **url**: file download link
* long **fileSize**: downloaded bytes
* boolean **resumable**: download link is resumable or not
* String **type**: file MIME
* int **chunks**: task chunks number
* double **percent**: downloaded file percent
* long **downloadLength**: size that will get from your sd card after it completely download
* String **saveAddress**: save file address
* boolean **priority**: true if task was high priority

Example:
```java
ReportStructure report = dm.singleDownloadStatus(taskToken);
```

----

It's a report method for returning the list of download task in same state that developers want.
```java
List DownloadManagerPro.downloadTasksInSameState(int state)
```

* int **state**: any download in it's life time across 6 state.
 * TaskState.INIT: task intruduce for library and gave you token back but it didn't started yet.
 * TaskState.READY: download task data fetch from its URL and it's ready to start.
 * TaskState.DOWNLOADING: download task in downloading process.
 * TaskState.PAUSED: download task in puase state. If in middle of downloading process internet disconnected; task goes to puase state and you can start it later
 * TaskState.DOWNLOAD_FINISHED: download task downloaded completely but their chunks did not rebuild.
 * TaskState.END: after rebuild download task chunks, task goes to this state and notified developer with OnDownloadCompleted(long taskToken) interface


Example:
```java
List<ReportStructure> report = dm.downloadTasksInSameState(TaskState.INIT);
```

-------

This method returns list of last completed Download tasks in "ReportStructure" style, developers can use it for notifying whether the task is completed or not.
```java
List DownloadManagerPro.lastCompletedTasks()
```

* return List<ReportStructure> : list of completed download from last called unNotifiedCheck() method till now.


Example:
```java
List<ReportStructure> completedDownloadTasks = dm.lastCompletedTasks();
```

--------------

This method checks all un notified tasks, so in another "lastCompletedDownloads" call ,completed task does not show up again. “lastCompletedDownloads”: Shows the list of latest completed downloads. Calling this method, all of the tasks that were shown in the previous report, will be eliminated from "lastCompletedDownloads"
```java
void DownloadManagerPro.unNotifiedCheck()
```
Example:
```java
dm.unNotifiedCheck()
```

----------


this method delete download task
```java
boolean DownloadManagerPro.delete(int token, boolean deleteTaskFile)
```
* int **token**: download task token
* boolean **deleteTaskFile**: deletes download task from database and set deleteTaskFile as true, then it goes to saved folder and delete that file.

*return **boolean** : if delete is successfully it returns true otherwise false

Example:
```java
dm.delete(12, false);
```

------------

This method closes database connection.

```java
void DownloadManagerPro.disConnectDB()
```

Example:
```java
dm.disConnectDb();
```
