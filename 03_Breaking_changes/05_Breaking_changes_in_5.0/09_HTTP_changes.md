## HTTP协议变更 HTTP changes

### Compressed HTTP requests are always accepted

Before 5.0, Elasticsearch accepted compressed HTTP requests only if the setting `http.compressed` was set to `true`. Elasticsearch accepts compressed requests now but will continue to send compressed responses only if `http.compressed` is set to `true`.
