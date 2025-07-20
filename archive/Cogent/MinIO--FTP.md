# MinIO FTP 断点续传

对于minio来说，使用minio官方的Java SDK和开启FTP都是不支持断点续传的。对于要实现http接口的断点续传，可以通过调用Amazon S3 REST API来实现，可以参考开源项目：https://gitee.com/Gary2016/minio-upload



本文是关于FTP断点续传。

## FTP断点续传方案

启动一个FTP服务器，此FTP支持断点续传。然后将FTP上传的文件同步到MinIO

同步的方式有两种：

- 通过定时任务扫描指定目录（ftp上传时也上传到指定目录），如果有更新时间大于Minio中最新对象的上传时间的，那么就说明应该将他同步到Minio上
- 提供一个按钮，给用户可以手动刷新，触发ftp指定文件夹下的“符合条件的”文件同步到Minio



## 实现

下面给出定时任务的代码，基本流程是：

1. 每一个minio object的信息我都在数据库中记录。我先从数据库中查询最新上传的minio的object的date
2. Java扫描ftp约定好上传的目录，如果文件的modify time 大于 minio object最新上传的 time，那么说明此文件应该同步到上传到minio上
3. 拿到这些应该上传到minio 的file，for循环以此上传即可。

代码如下：

```java
    @Value("${ftpDirPath}")
    private String ftpDirPath;

    MinioClient minioClient;
    @PostConstruct
    void init() {
        minioClient = MinioClient.builder()
                .endpoint("http://" + minioAddress)
                .credentials(minioAccessKey, minioSecretKey)
                .build();
    }

	@SneakyThrows
    @Scheduled(fixedDelay = 60 * 60 * 1000, initialDelay = 10 * 60 * 1000)
    public void checkFtpServer() {
        log.info("Check ftp server /opt/ftp directory have new file upload.");
        // get latest time form minio_object table. if file update time is bigger than minio_object latest time.
        // then we should upload the file to minio.
        // notice: if i use update time, I should ensure linux date is same as database date (notice same timezone!!)
        MinioObjectDO latestDateDO = minioObjectDao.getLatestDate();
        long latestTime;
        if (latestDateDO == null) {
            latestTime = 0L;
        } else {
            latestTime = latestDateDO.getUploadTime().getTime();
        }
        LinkedList<File> needUploadToMinioFile = getDirAllFileFilterWithLastModify(ftpDirPath, latestTime);

        // upload file to minio
        for (File file : needUploadToMinioFile) {
            InputStream initialStream = Files.newInputStream(file.toPath());
            minioClient.putObject(
                    PutObjectArgs.builder().bucket("cogent").object(removeFtpPath(file.getAbsolutePath())).stream(
                                    initialStream, -1, 10485760)
                            .build());

            initialStream.close();
            log.info("upload file: {} successfully", file.getName());
        }
    }

    private String removeFtpPath(String absolutePath) {
        return absolutePath.substring(ftpDirPath.length() + 1);
    }

    private LinkedList<File> getDirAllFileFilterWithLastModify(String ftpDirPath,long latestTime) {
        File dirPath = new File(ftpDirPath);
        // 2023-12-01 时间戳
        long time = latestTime;
        LinkedList<File> filteredFiles = new LinkedList<>();
        process(dirPath, filteredFiles, time);
        return filteredFiles;
    }

    private void process(File dirPath, LinkedList<File> filteredFiles, long time) {
        File[] files = dirPath.listFiles();
        for (File file : files) {
            if (file.isDirectory()) {
                process(file, filteredFiles, time);
            } else {
                if (file.lastModified() >= time) {
                    filteredFiles.add(file);
                }
            }
        }
    }
```



这里只放定时任务的。

## issue

### 关于断点续传只上传了部分文件

ftp断点续传只上传了部分文件，然后这部分文件被同步到了minio上。这其实没有问题，如果ftp接着上传剩下的文件，那么文件的updateTime就会修改，那么下次再同步时会再次上传此文件，对于同名的object，minio进行覆盖。