---
layout: post
title: "yarn 日志聚集权限警告"
date: 2015-08-24 18:04:00 +0800
comments: true
categories: hadoop
---

### 问题描述
集群中所有的nodemanager节点机器总是会报出警告信息
> 警告信息：
2015-07-29 00:44:57,785 WARN org.apache.hadoop.yarn.server.nodemanager.containermanager.logaggregation.LogAggregationService: Remote Root Log Dir [/yarn-logs] already exist, but with incorrect permissions. Expected: [rwxrwxrwt], Found: [rwxr-xr-x]. The cluster may have problems with multiple users.

从警告的内容来看，似乎是目录的权限不匹配导致的，为了防止这个警告产生对集群的影响，于是排查本异常产生的原因。
<!-- more -->
### 问题原因
产生警告的类为 LogAggregationService.java , 其中产生警告的代码为
``` java
 void verifyAndCreateRemoteLogDir(Configuration conf) {
  // Checking the existence of the TLD
  FileSystem remoteFS = null;
  try {
    remoteFS = getFileSystem(conf);
  } catch (IOException e) {
    throw new YarnRuntimeException("Unable to get Remote FileSystem instance", e);
  }
  boolean remoteExists = true;
  try {
    FsPermission perms =
        remoteFS.getFileStatus(this.remoteRootLogDir).getPermission();
    if (!perms.equals(TLDIR_PERMISSIONS)) {
      LOG.warn("Remote Root Log Dir [" + this.remoteRootLogDir
          + "] already exist, but with incorrect permissions. "
          + "Expected: [" + TLDIR_PERMISSIONS + "], Found: [" + perms
          + "]." + " The cluster may have problems with multiple users.");
    }
  } catch (FileNotFoundException e) {
    remoteExists = false;
  } catch (IOException e) {
    throw new YarnRuntimeException(
        "Failed to check permissions for dir ["
            + this.remoteRootLogDir + "]", e);
  }
  if (!remoteExists) {
    LOG.warn("Remote Root Log Dir [" + this.remoteRootLogDir
        + "] does not exist. Attempting to create it.");
    try {
      Path qualified =
          this.remoteRootLogDir.makeQualified(remoteFS.getUri(),
              remoteFS.getWorkingDirectory());
      remoteFS.mkdirs(qualified, new FsPermission(TLDIR_PERMISSIONS));
      remoteFS.setPermission(qualified, new FsPermission(TLDIR_PERMISSIONS));
    } catch (IOException e) {
      throw new YarnRuntimeException("Failed to create remoteLogDir ["
          + this.remoteRootLogDir + "]", e);
    }
  }
}
```

代码中的第11-17行中就是这个警告产生的过程。从代码不难看出，这个警告不会对系统运行产生影响，也不会给系统造成多大的负担。
从代码的逻辑中，我们可以看出hadoop希望我们将聚合日志的目录（就是配置属性yarn.nodemanager.remote-app-log-dir指定的目录）设置为权限01777（其中1权限代码sticky bit），权限参考HDFS 权限说明 。而实际我们的集群因为保证完全的用户隔离和写安全，将日志聚合目录修改为了755权限（我们会单独为每个新用户创建相应的聚合目录），因此导致hadoop期望的权限和我们设置的权限不匹配，从而导致了问题的产生。

### 解决方法
解决方法1： 如果对系统的安全性要求没有特别的要求，可以完全将日志聚合目录的权限按hadoop的要求修改为01777权限，这个问题自然解决。

解决方法2：在创建新用户的情况下，可以手动给每个用户创建对应的聚合目录，也不会影响聚合功能的使用。