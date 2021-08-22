I'm running Docker 20.10.8 with the sismics/docs:v1.9 image. Using the default admin account I can't seem to use the web UI to upload a PDF file larger than about 1 MB.

Logs for a successful upload via `docker logs`:

```
 22 Aug 2021 08:18:12,591 INFO com.sismics.docs.core.util.FileUtil.startProcessingFile(FileUtil.java:221) Processing started for file: 8e2d26f3-0d6d-4a48-8790-606f39194c8c
 22 Aug 2021 08:18:12,613 INFO com.sismics.docs.core.listener.async.FileProcessingAsyncListener.on(FileProcessingAsyncListener.java:53) File created event: FileCreatedAsyncEvent{fileId=8e2d26f3-0d6d-4a48-8790-606f39194c8c, language=null}
 22 Aug 2021 08:18:14,105 INFO com.sismics.docs.core.listener.async.FileProcessingAsyncListener.extractContent(FileProcessingAsyncListener.java:176) Start extracting content from: File{id=8e2d26f3-0d6d-4a48-8790-606f39194c8c, name=netcat_cheat_sheet_v1.pdf}
 22 Aug 2021 08:18:14,278 INFO com.sismics.docs.core.listener.async.FileProcessingAsyncListener.extractContent(FileProcessingAsyncListener.java:182) File content extracted in 178ms: 8e2d26f3-0d6d-4a48-8790-606f39194c8c
 22 Aug 2021 08:18:14,336 INFO com.sismics.docs.core.util.FileUtil.endProcessingFile(FileUtil.java:231) Processing ended for file: 8e2d26f3-0d6d-4a48-8790-606f39194c8c
```

Trying to upload a larger file (1.6 MB) results in the following error on the web UI:


And the logs show no change, no indication of the failed upload. Dev tools in Firefox say I'm getting a `413 Request Entity Too Large` error, which made me suspect the problem was instead nginx.

The (nginx docs)[http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size] confirmed that the `client_max_body_size` directive defaults to just 1 MB.
