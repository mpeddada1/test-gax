# test-gax

This project is to reproduce an incompatibility issue with `com.google.cloud:google-cloud-compute:1.12.0` and `org.slf4j:jcl-over-slf4j:1.7.36`. 

1) `java -version`
```
openjdk version "11.0.16" 2022-07-19
OpenJDK Runtime Environment GraalVM CE 22.2.0 (build 11.0.16+8-jvmci-22.2-b06)
OpenJDK 64-Bit Server VM GraalVM CE 22.2.0 (build 11.0.16+8-jvmci-22.2-b06, mixed mode, sharing)
```
2) `mvn clean install -DskipTests`
3) `cd child-project`.
4) `mvn package -Pnative`
5) See the following error:
```
========================================================================================================================
GraalVM Native Image: Generating 'child-project' (executable)...
========================================================================================================================
[1/7] Initializing...                                                                                   (11.7s @ 0.54GB)
 Version info: 'GraalVM 22.2.0 Java 11 CE'
 Java version info: '11.0.16+8-jvmci-22.2-b06'
 C compiler: gcc (linux, x86_64, 11.3.0)
 Garbage collector: Serial GC
 5 user-specific feature(s)
 - com.google.api.gax.grpc.nativeimage.GrpcNettyFeature
 - com.google.api.gax.grpc.nativeimage.ProtobufMessageFeature
 - com.google.api.gax.nativeimage.GoogleJsonClientFeature
 - com.google.api.gax.nativeimage.OpenCensusFeature
 - com.oracle.svm.thirdparty.gson.GsonFeature
[2/7] Performing analysis...  [*]                                                                       (10.1s @ 1.05GB)
   4,968 (81.30%) of  6,111 classes reachable
   7,623 (61.16%) of 12,465 fields reachable
  27,962 (59.19%) of 47,245 methods reachable
     403 classes, 1,029 fields, and 6,541 methods registered for reflection
       1 native library: stdc++

Fatal error: com.oracle.graal.pointsto.util.AnalysisError$ParsingError: Error encountered while parsing io.grpc.netty.shaded.io.netty.channel.nio.AbstractNioChannel.<init>(io.grpc.netty.shaded.io.netty.channel.Channel, java.nio.channels.SelectableChannel, int) 
Parsing context:
   at io.grpc.netty.shaded.io.netty.channel.nio.AbstractNioChannel.<init>(AbstractNioChannel.java:80)
   at io.grpc.netty.shaded.io.netty.channel.nio.AbstractNioMessageChannel.<init>(AbstractNioMessageChannel.java:42)

	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.util.AnalysisError.parsingError(AnalysisError.java:152)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlow.createFlowsGraph(MethodTypeFlow.java:104)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlow.ensureFlowsGraphCreated(MethodTypeFlow.java:83)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlow.getOrCreateMethodFlowsGraph(MethodTypeFlow.java:65)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.typestate.DefaultSpecialInvokeTypeFlow.onObservedUpdate(DefaultSpecialInvokeTypeFlow.java:61)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.TypeFlow.update(TypeFlow.java:558)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.PointsToAnalysis$1.run(PointsToAnalysis.java:635)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.util.CompletionExecutor.executeCommand(CompletionExecutor.java:193)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.util.CompletionExecutor.lambda$executeService$0(CompletionExecutor.java:177)
	at java.base/java.util.concurrent.ForkJoinTask$RunnableExecuteAction.exec(ForkJoinTask.java:1426)
	at java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:290)
	at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(ForkJoinPool.java:1020)
	at java.base/java.util.concurrent.ForkJoinPool.scan(ForkJoinPool.java:1656)
	at java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1594)
	at java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:183)
Caused by: com.oracle.graal.pointsto.constraints.UnsupportedFeatureException: No instances of org.slf4j.impl.JDK14LoggerAdapter are allowed in the image heap as this class should be initialized at image runtime. Object has been initialized by the io.grpc.netty.shaded.io.netty.channel.nio.AbstractNioChannel class initializer with a trace: 
 	at org.slf4j.impl.JDK14LoggerAdapter.<init>(JDK14LoggerAdapter.java:56)
	at org.slf4j.impl.JDK14LoggerFactory.getLogger(JDK14LoggerFactory.java:67)
	at org.slf4j.LoggerFactory.getLogger(LoggerFactory.java:363)
	at io.grpc.netty.shaded.io.netty.util.internal.logging.Slf4JLoggerFactory.newInstance(Slf4JLoggerFactory.java:49)
	at io.grpc.netty.shaded.io.netty.util.internal.logging.InternalLoggerFactory.getInstance(InternalLoggerFactory.java:134)
	at io.grpc.netty.shaded.io.netty.util.internal.logging.InternalLoggerFactory.getInstance(InternalLoggerFactory.java:127)
	at io.grpc.netty.shaded.io.netty.channel.nio.AbstractNioChannel.<clinit>(AbstractNioChannel.java:51)
.  To fix the issue mark org.slf4j.impl.JDK14LoggerAdapter for build-time initialization with --initialize-at-build-time=org.slf4j.impl.JDK14LoggerAdapter or use the the information from the trace to find the culprit and --initialize-at-run-time=<culprit> to prevent its instantiation.

	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.classinitialization.ClassInitializationFeature.checkImageHeapInstance(ClassInitializationFeature.java:140)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.meta.AnalysisUniverse.replaceObject(AnalysisUniverse.java:583)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.ameta.AnalysisConstantReflectionProvider.replaceObject(AnalysisConstantReflectionProvider.java:257)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.ameta.AnalysisConstantReflectionProvider.interceptValue(AnalysisConstantReflectionProvider.java:228)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.ameta.AnalysisConstantReflectionProvider.readValue(AnalysisConstantReflectionProvider.java:105)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.ameta.AnalysisConstantReflectionProvider.readFieldValue(AnalysisConstantReflectionProvider.java:84)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.util.ConstantFoldUtil$1.readValue(ConstantFoldUtil.java:55)
	at jdk.internal.vm.compiler/org.graalvm.compiler.core.common.spi.JavaConstantFieldProvider.readConstantField(JavaConstantFieldProvider.java:78)
	at org.graalvm.nativeimage.builder/com.oracle.svm.hosted.ameta.AnalysisConstantFieldProvider.readConstantField(AnalysisConstantFieldProvider.java:72)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.util.ConstantFoldUtil.tryConstantFold(ConstantFoldUtil.java:51)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.java.LoadFieldNode.asConstant(LoadFieldNode.java:178)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.java.LoadFieldNode.canonical(LoadFieldNode.java:144)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.java.LoadFieldNode.canonical(LoadFieldNode.java:135)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.java.LoadFieldNode.canonical(LoadFieldNode.java:72)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.spi.Canonicalizable$Unary.canonical(Canonicalizable.java:101)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.SimplifyingGraphDecoder.canonicalizeFixedNode(SimplifyingGraphDecoder.java:214)
	at jdk.internal.vm.compiler/org.graalvm.compiler.replacements.PEGraphDecoder.canonicalizeFixedNode(PEGraphDecoder.java:1572)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.phases.InlineBeforeAnalysisGraphDecoder.canonicalizeFixedNode(InlineBeforeAnalysis.java:192)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.SimplifyingGraphDecoder.handleFixedNode(SimplifyingGraphDecoder.java:193)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.GraphDecoder.processNextNode(GraphDecoder.java:821)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.phases.InlineBeforeAnalysisGraphDecoder.processNextNode(InlineBeforeAnalysis.java:240)
	at jdk.internal.vm.compiler/org.graalvm.compiler.nodes.GraphDecoder.decode(GraphDecoder.java:548)
	at jdk.internal.vm.compiler/org.graalvm.compiler.replacements.PEGraphDecoder.decode(PEGraphDecoder.java:833)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.phases.InlineBeforeAnalysis.decodeGraph(InlineBeforeAnalysis.java:98)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlowBuilder.parse(MethodTypeFlowBuilder.java:176)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlowBuilder.apply(MethodTypeFlowBuilder.java:343)
	at org.graalvm.nativeimage.pointsto/com.oracle.graal.pointsto.flow.MethodTypeFlow.createFlowsGraph(MethodTypeFlow.java:93)
	... 13 more
------------------------------------------------------------------------------------------------------------------------
                        1.4s (6.0% of total time) in 20 GCs | Peak RSS: 3.40GB | CPU load: 5.64
========================================================================================================================
Failed generating 'child-project' after 22.1s.
Error: Image build request failed with exit status 1
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  24.203 s
[INFO] Finished at: 2022-08-30T00:44:44-04:00
[INFO] ------------------------------------------------------------------------
```

Calling `mvn dependency:tree` results in:
```
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ child-project ---
[INFO] com.example:child-project:jar:1.0-SNAPSHOT
[INFO] +- org.slf4j:slf4j-jdk14:jar:1.7.36:compile
[INFO] |  \- org.slf4j:slf4j-api:jar:1.7.36:compile
[INFO] +- org.slf4j:jcl-over-slf4j:jar:1.7.36:compile
[INFO] +- com.google.cloud:google-cloud-compute:jar:1.12.1-SNAPSHOT:compile
[INFO] |  +- com.google.api:api-common:jar:2.2.1:compile
[INFO] |  +- com.google.auto.value:auto-value-annotations:jar:1.9:compile
[INFO] |  +- com.google.api.grpc:proto-google-cloud-compute-v1:jar:1.12.1-SNAPSHOT:compile
[INFO] |  +- com.google.guava:guava:jar:31.1-jre:compile
[INFO] |  +- com.google.guava:failureaccess:jar:1.0.1:compile
[INFO] |  +- com.google.guava:listenablefuture:jar:9999.0-empty-to-avoid-conflict-with-guava:compile
[INFO] |  +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] |  +- org.checkerframework:checker-qual:jar:3.23.0:compile
[INFO] |  +- com.google.errorprone:error_prone_annotations:jar:2.14.0:compile
[INFO] |  +- com.google.j2objc:j2objc-annotations:jar:1.3:compile
[INFO] |  +- com.google.api:gax:jar:2.18.7:compile
[INFO] |  +- com.google.auth:google-auth-library-credentials:jar:1.8.1:compile
[INFO] |  +- com.google.auth:google-auth-library-oauth2-http:jar:1.8.1:compile
[INFO] |  +- com.google.api:gax-grpc:jar:2.18.7:compile
[INFO] |  +- io.grpc:grpc-netty-shaded:jar:1.48.0:runtime
[INFO] |  +- io.perfmark:perfmark-api:jar:0.25.0:runtime
[INFO] |  +- io.grpc:grpc-core:jar:1.48.0:runtime
[INFO] |  +- com.google.android:annotations:jar:4.1.1.4:runtime
[INFO] |  +- org.codehaus.mojo:animal-sniffer-annotations:jar:1.21:runtime
[INFO] |  +- com.google.api:gax-httpjson:jar:0.103.7:compile
[INFO] |  +- com.google.code.gson:gson:jar:2.9.1:compile
[INFO] |  +- com.google.http-client:google-http-client:jar:1.42.2:compile
[INFO] |  +- commons-codec:commons-codec:jar:1.15:compile
[INFO] |  +- io.opencensus:opencensus-api:jar:0.31.1:compile
[INFO] |  +- io.grpc:grpc-context:jar:1.48.0:compile
[INFO] |  +- io.opencensus:opencensus-contrib-http-util:jar:0.31.1:compile
[INFO] |  +- com.google.http-client:google-http-client-gson:jar:1.42.2:compile
[INFO] |  +- com.google.protobuf:protobuf-java-util:jar:3.21.4:compile
[INFO] |  +- org.threeten:threetenbp:jar:1.6.0:compile
[INFO] |  +- com.google.api.grpc:proto-google-common-protos:jar:2.9.2:compile
[INFO] |  +- com.google.protobuf:protobuf-java:jar:3.21.4:compile
[INFO] |  \- javax.annotation:javax.annotation-api:jar:1.3.2:compile
[INFO] \- com.google.cloud:google-cloud-storage:jar:2.11.3:compile
[INFO]    +- com.google.http-client:google-http-client-jackson2:jar:1.42.2:compile
[INFO]    +- com.google.api-client:google-api-client:jar:2.0.0:compile
[INFO]    +- com.google.oauth-client:google-oauth-client:jar:1.34.1:compile
[INFO]    +- com.google.apis:google-api-services-storage:jar:v1-rev20220705-2.0.0:compile
[INFO]    +- com.google.cloud:google-cloud-core:jar:2.8.6:compile
[INFO]    +- com.google.cloud:google-cloud-core-http:jar:2.8.6:compile
[INFO]    +- com.google.http-client:google-http-client-appengine:jar:1.42.2:compile
[INFO]    +- com.google.api.grpc:proto-google-iam-v1:jar:1.5.2:compile
[INFO]    \- com.fasterxml.jackson.core:jackson-core:jar:2.13.3:compile

```
gax-grpc brings in some unused dependencies (such as `netty-shaded`) which conflict with other libraries like slf4j. 
gax-grpc is only needed for the purpose of bringing native-image configurations so the dependencies it brings in are unnecessary. 

To resolve the failure:
Modify version of compute to `1.12.1-SNAPSHOT` (which contains the exclusions) and re-run. 
