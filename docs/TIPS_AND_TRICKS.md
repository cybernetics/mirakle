# Tips and Tricks

- [Speed up Android build by breaking remote execution on `package` task](#speed-up-android-build-by-breaking-remote-execution-on-package-task)


### Speed up Android build by breaking remote execution on `package` task 

The main result of Android application build is `.apk` file which contains all the code and resources of the application.
 `.apk` file is a zip archive that can't be downloaded from remote machine by rsync incrementally.
 Making just a small fix to codebase or adding new resource will lead to the need of downloading the whole file the size of which can be quite large.
 
The idea of breaking remote building process on `package` task is to stop remote build one step before archiving all the application files into `.apk` file 
and let rsync incrementally download only these files that changed during the build.
One line fix usually affects one `.dex` file whose size is significantly less than size of whole `.apk` file.
 
Mirakle will download all the inputs of `package` task and execute it by Gradle on local machine.        
      
```groovy
initscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "com.instamotor:mirakle:1.4.0-RC-1"
    }
}
 
apply plugin: MirakleBreakMode // <- 
 
rootProject {
    mirakle {
        host "your_remote_machine"
        breakOnTasks += ["package"] // this is regex pattern that matches all build flavour variations of package task.  
        excludeRemote += ["build/tmp", "build/generated", "build/intermediates/*", "build/kotlin"]
        rsyncFromRemoteArgs += ["-i", "-c", "--compress-level=2"] 
    }
}
```