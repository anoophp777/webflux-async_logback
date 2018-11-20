# Aysnchonous logging using logback for both reactive and non-reactive endpoints

## Overview

Logging is helpful but can take a toll on the system, especially if there is a lot of load on the system, which leads to higher log load. 

Asynchonous logging is far beneficial in such cases. 

Here is a comparison. 

https://blog.takipi.com/how-to-instantly-improve-your-java-logging-with-7-logback-tweaks/

I quote glytching from stackoverflow:

(https://stackoverflow.com/questions/47372092/when-not-to-use-asyncappender-in-logback-by-default)

The AsyncAppender acts as a dispatcher to another appender. It buffers log events and dispatches them to, say, a FileAppender or a ConsoleAppender etc.

### Why use the AsyncAppender?

>The AsyncAppender buffers log events, allowing your application code to move on rather than wait for the logging sub system to complete a write. This can improve your application's responsiveness in cases where the underying appender is slow to respond e.g. a database or a file system which may be prone to contention.

### Why not make it the default behaviour?

>The AsyncAppender cannot write to a file or console or a database or a socket etc. Instead, it just delegates log events to an appender which can do that. Without the underlying appender the AsyncAppender is, effectively, a no-op.
The buffer of log events sits on your application's heap; this is a potential resource leak. If the buffer builds more quickly than it can be drained then the buffer will consume resources which your application might want to use.
The AsyncAppender's need for configuration to balance the competing demands of no-loss and resource leakage and to handle on-shutdown draining of its buffer means that it is more complicated to manage and to reason about than simply using synchronous writes. So, on the basis of preferring simplicity over complexity it makes sense for Logback's default write strategy to be synchronous.

The AsyncAppender exposes configuration levers which you can use to address the potential resource leakage. For example:

- You can increase the buffer capacity
- You can instruct Logback to drop events once the buffer reaches maximum capcacity
- You can control what types of events are discarded; drop TRACE events before ERROR events etc

The AsyncAppender also exposes configuration levers which you can use to limit (though not eliminate) the loss of events during application shutdown.

However, it remains true that the simplest safest way of ensuring that log events are successfully writtten is to write them synchronously. The AsyncAppender should only be considered when you have a proven issue where writing to an appender materially affects your application responsiveness / throughput. 
  
### How to run?

cd to webflux-async_logback and override the logging.file in application.properties.

Run
  
	mvn spring-boot:run
	
Once the application is booted up, you can test the reactive endpoint by curling:

	curl http://localhost:7070/byPriceReactive?maxPrice=1
	
You will notice logback is printing in the console and the log file the following:


	2018-11-19 12:51:38.381 [{X-B3-SpanId=8b1e7c40420e8324, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, spanExportable=true, spanId=8b1e7c40420e8324, traceId=5bf264826fe743518b1e7c40420e8324}] DEBUG ahallim-1ef960 --- [nio-7070-exec-2] a.h.w.RestaurantController               : tracer: brave.opentracing.BraveTracer@517bafcb
	2018-11-19 12:51:38.382 [{X-B3-SpanId=8b1e7c40420e8324, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, spanExportable=true, spanId=8b1e7c40420e8324, traceId=5bf264826fe743518b1e7c40420e8324}]  INFO ahallim-1ef960 --- [nio-7070-exec-2] a.h.w.RestaurantController               : active span: brave.opentracing.BraveSpan@1c1c76cd
	2018-11-19 12:51:38.382 [{X-B3-SpanId=8b1e7c40420e8324, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, spanExportable=true, spanId=8b1e7c40420e8324, traceId=5bf264826fe743518b1e7c40420e8324}] DEBUG ahallim-1ef960 --- [nio-7070-exec-2] a.h.w.RestaurantController               : starting statement
	2018-11-19 12:51:38.393 [{X-B3-ParentSpanId=8b1e7c40420e8324, X-B3-SpanId=9cbbfc8112535548, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, parentId=8b1e7c40420e8324, spanExportable=true, spanId=9cbbfc8112535548, traceId=5bf264826fe743518b1e7c40420e8324}] DEBUG ahallim-1ef960 --- [nio-7070-exec-2] a.h.w.RestaurantService                  : inside byPrice Reactive
	2018-11-19 12:51:39.423 [{X-B3-SpanId=8b1e7c40420e8324, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, spanExportable=true, spanId=8b1e7c40420e8324, traceId=5bf264826fe743518b1e7c40420e8324}] DEBUG ahallim-1ef960 --- [     parallel-1] a.h.w.RestaurantController               : found restaurant McDonalds for $1.0
	2018-11-19 12:51:39.423 [{X-B3-SpanId=8b1e7c40420e8324, X-B3-TraceId=5bf264826fe743518b1e7c40420e8324, X-Span-Export=true, spanExportable=true, spanId=8b1e7c40420e8324, traceId=5bf264826fe743518b1e7c40420e8324}] DEBUG ahallim-1ef960 --- [     parallel-1] a.h.w.RestaurantController               : done!
 

To test the traditional non-reactive endpoint: 
	
	curl http://localhost:7070/byPriceMVC?maxPrice=1

You will see in the logs:

	2018-11-19 12:54:41.117 [{X-B3-SpanId=ead6d4e8397e7dd0, X-B3-TraceId=5bf265397a815f2cead6d4e8397e7dd0, X-Span-Export=true, spanExportable=true, spanId=ead6d4e8397e7dd0, traceId=5bf265397a815f2cead6d4e8397e7dd0}] DEBUG ahallim-1ef960 --- [nio-7070-exec-6] a.h.w.RestaurantController               : tracer: brave.opentracing.BraveTracer@43c6e747
	2018-11-19 12:54:41.118 [{X-B3-SpanId=ead6d4e8397e7dd0, X-B3-TraceId=5bf265397a815f2cead6d4e8397e7dd0, X-Span-Export=true, spanExportable=true, spanId=ead6d4e8397e7dd0, traceId=5bf265397a815f2cead6d4e8397e7dd0}]  INFO ahallim-1ef960 --- [nio-7070-exec-6] a.h.w.RestaurantController               : active span: brave.opentracing.BraveSpan@787945bc
	2018-11-19 12:54:41.118 [{X-B3-SpanId=ead6d4e8397e7dd0, X-B3-TraceId=5bf265397a815f2cead6d4e8397e7dd0, X-Span-Export=true, spanExportable=true, spanId=ead6d4e8397e7dd0, traceId=5bf265397a815f2cead6d4e8397e7dd0}] DEBUG ahallim-1ef960 --- [nio-7070-exec-6] a.h.w.RestaurantController               : starting statement
	2018-11-19 12:54:41.118 [{X-B3-ParentSpanId=ead6d4e8397e7dd0, X-B3-SpanId=521cd352679aee4f, X-B3-TraceId=5bf265397a815f2cead6d4e8397e7dd0, X-Span-Export=true, parentId=ead6d4e8397e7dd0, spanExportable=true, spanId=521cd352679aee4f, traceId=5bf265397a815f2cead6d4e8397e7dd0}] DEBUG ahallim-1ef960 --- [nio-7070-exec-6] a.h.w.RestaurantService                  : inside byPrice MVC
	
All this is happening asynchronously. You can put a load on the system and play around with the settings. 