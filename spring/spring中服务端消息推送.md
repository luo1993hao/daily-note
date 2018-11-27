#### 步骤
1. 注册监听器
2. 发送消息推送
##### 细节
核心就是SSeEmitter 这个对象
在注册的时候。做好该对象各种异常处理。
```
  正常结束回调
 
        emitter.onCompletion(() -> boxEmitters.remove(emitter));
        // 超时结束回调
        emitter.onTimeout(() -> {
            log.debug("监听者连接超时关闭！");
            emitter.complete();
            boxEmitters.remove(emitter);
        });
        // 异常结束回调
        emitter.onError(t -> {
            log.debug("监听者异常关闭！");
            if (log.isTraceEnabled()) {
                log.trace(t.getMessage(), t);
            }
            boxEmitters.remove(emitter);
        });
```
调用的时候使用SseEmitter的send方法即可。如果异常remove掉就OK
```
  try {
                emitter.send(message, MediaType.TEXT_PLAIN);
            } catch (IOException e) {
                emitter.complete();
                this.emitters.remove(emitter);
                log.warn(e.getMessage(), e);
            }
```
构建什么样的数据结构对象都无所谓。map,list或者是一个对象。只要在整个调用过程中。SseEmitter对象贯穿其中就行