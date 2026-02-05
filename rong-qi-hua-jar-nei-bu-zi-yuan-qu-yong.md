---
description: 容器化
---

# 容器化Jar內部資源取用

當我們使用了容器化部署ＡＰＩ服務，若要取用Jar包內檔案資源的話，因為容器化的關係，所以無法正常定位到正確的檔案。

因此解法為：

先透過InputStream 進行讀取Jar包內的檔案資源，再行寫出該檔案到Jar包外，提供程式讀取。

```java
 InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("Jar檔內資源路徑");
                    //取得jar執行路徑
                    String path = getClass().getProtectionDomain().getCodeSource().getLocation().getPath();
                    //修改路徑到目錄下
                    path = path.replace("wise-common-1.0.0-snapshot.jar", "") + kafkaMatcher.group(1);
                    LOGGER.info("ＳＳＬ路徑："+ path);
                    //寫入檔案
                    File targetFile = new File(path);
                    java.nio.file.Files.copy(
                            inputStream,
                            targetFile.toPath(),
                            StandardCopyOption.REPLACE_EXISTING);
                    IOUtils.closeQuietly(inputStream);
                    //改寫變數路徑到重新輸出檔案位置
                    entry.setValue(path);

```

