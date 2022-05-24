# Sink

这个 sink 的意思也不一定非得说成要把数据存储到某个地方去。其实官网用的 Connector 来形容要去的地方更合适，这个 Connector 可以有 MySQL、ElasticSearch、Kafka、Cassandra RabbitMQ 等。

可以看到有 Kafka、ElasticSearch、Socket、RabbitMQ、JDBC、Cassandra POJO、File、Print 等 Sink 的方式。

### SinkFunction

从上图可以看到 SinkFunction 接口有 invoke 方法，它有一个 RichSinkFunction 抽象类。

上面的那些自带的 Sink 可以看到都是继承了 RichSinkFunction 抽象类，实现了其中的方法，那么我们要是自己定义自己的 Sink 的话其实也是要按照这个套路来做的。

自定义Sink继承RichSinkFunction

```
@PublicEvolving
public class PrintSinkFunction<IN> extends RichSinkFunction<IN> {
	private static final long serialVersionUID = 1L;

	private static final boolean STD_OUT = false;
	private static final boolean STD_ERR = true;

	private boolean target;
	private transient PrintStream stream;
	private transient String prefix;

	/**
	 * Instantiates a print sink function that prints to standard out.
	 */
	public PrintSinkFunction() {}

	/**
	 * Instantiates a print sink function that prints to standard out.
	 *
	 * @param stdErr True, if the format should print to standard error instead of standard out.
	 */
	public PrintSinkFunction(boolean stdErr) {
		target = stdErr;
	}

	public void setTargetToStandardOut() {
		target = STD_OUT;
	}

	public void setTargetToStandardErr() {
		target = STD_ERR;
	}

	@Override
	public void open(Configuration parameters) throws Exception {
		super.open(parameters);
		StreamingRuntimeContext context = (StreamingRuntimeContext) getRuntimeContext();
		// get the target stream
		stream = target == STD_OUT ? System.out : System.err;

		// set the prefix if we have a >1 parallelism
		prefix = (context.getNumberOfParallelSubtasks() > 1) ?
				((context.getIndexOfThisSubtask() + 1) + "> ") : null;
	}

	@Override
	public void invoke(IN record) {
		if (prefix != null) {
			stream.println(prefix + record.toString());
		}
		else {
			stream.println(record.toString());
		}
	}

	@Override
	public void close() {
		this.stream = null;
		this.prefix = null;
	}

	@Override
	public String toString() {
		return "Print to " + (target == STD_OUT ? "System.out" : "System.err");
	}
}
```

