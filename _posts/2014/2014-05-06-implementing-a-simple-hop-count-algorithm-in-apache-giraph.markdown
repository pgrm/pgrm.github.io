---
title: Implementing a simple hop-count algorithm in Apache Giraph
date: 2014-05-06 00:00:00 Z
tags:
- Apache-Giraph
- graph-processing
- hop-count
---

Hi, this is my second post after [First steps with Apache-Giraph](http://peter.grman.at/first-steps-with-apache-giraph/) on [Apache Giraph](https://giraph.apache.org/). I'll be writing a little bit about how you can use complex types in messages and in the vertex-state. To learn this I wrote a simple hop-count algorithm, you can find the full source code in a [branch of my Apache Giraph fork on GitHub](https://github.com/pgrm/giraph/blob/simple-hop-computation/giraph-examples/src/main/java/org/apache/giraph/examples).

Vertex-logic
--------

Ok, let's first have a look at the vertex computation class, [SimpleHopsComputation](https://github.com/pgrm/giraph/blob/simple-hop-computation/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleHopsComputation.java). In the `compute`-method you can see the types of the messages and the internal vertex state (`SimpleHopsMessage`, `SimpleHopsVertexValue`). You can also see, that the computation is kicked-off by asking all neighbors for the number of hops towards other vertices.

{% highlight java %}
public void compute(
        Vertex<LongWritable, SimpleHopsVertexValue, NullWritable> vertex,
        Iterable<SimpleHopsMessage> messages) throws IOException {
  this.vertex = vertex;

  if (super.getSuperstep() == 0) {
    askForNumberOfHops();
  }
  processMessages(messages);

  if (isTimeToStop()) {
    this.vertex.voteToHalt();
  }
}

/**
 * Each vertex has a list of vertices to which it wants to calculate the
 * number of hops. It does that by sending a message to all its neighbors.
 */
private void askForNumberOfHops() {
  for (Edge<LongWritable, NullWritable> edge : this.vertex.getEdges()) {
    if (isLoopBack(edge)) {
      continue; // skip loop back to current vertex
    }

    for (Long vertexId : this.vertex.getValue().getVertices()) {
      super.sendMessage(new LongWritable(edge.getTargetVertexId().get()),
              new SimpleHopsMessage(this.vertex.getId().get(), vertexId));
    }
  }
}
{% endhighlight %}

After that, a lot of messages are floating around which need to be processed. A message can be processed in 3 ways:

1. The current vector **is not** the final destination of the message. In this case, we will just forward it to all other neighbors.
2. The current vector **is** the final destination of the message. In this case, the vector sends the message back to its origin, so that this node can get the hop-count between the two vertices.
3. The message already has reached its final destination and came back. In this case we only need to update the number of hops towards the other vector in our internal state and it is done.

{% highlight java %}
/**
 * If the message wasn't sent to the current vertex, we will forward it to
 * all our neighbors.
 *
 * @param message the received message which will be forwarded.
 */
private void forward(SimpleHopsMessage message) {
  for (Edge<LongWritable, NullWritable> edge : this.vertex.getEdges()) {
    if (isLoopBack(edge)) {
      continue; // skip loop back to current vertex
    }

    super.sendMessage(edge.getTargetVertexId(), message);
  }
}

/**
 * If a messages has finally reached its destination and it's the current
 * vertex, than set the destinationFound-flag and send it back to the source.
 *
 * @param message the message which needs an answer
 */
private void reply(SimpleHopsMessage message) {
  message.setDestinationFound();
  super.sendMessage(new LongWritable(message.getSourceId()), message);
}
{% endhighlight %}

Vertex- & Message-data
---------------

Working with complex data-types in Apache Giraph is actually easy, you only need to consider two main things:

1. Implement the `org.apache.hadoop.io.Writable` Interface
2. Always have a public constructor without any parameters. This is necessary, so that Giraph is able to create your object via reflections.

### Message-data

Let's first look at the message-data type, [SimpleHopsMessage](https://github.com/pgrm/giraph/blob/simple-hop-computation/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleHopsMessage.java), because it is simpler.

{% highlight java %}
public class SimpleHopsMessage implements Writable {
  /** the id of the vertex initiating this message. */
  private long sourceId;
  /** the id of the vertex of the final destination of this message. */
  private long destinationId;
  /** the current number of hops, the message had to take. */
  private int hopsCount = 0;
  /** Flag, if the destination has been found, and this message is an answer. */
  private boolean destinationFound = false;

  /** Default constructor for reflection */
  public SimpleHopsMessage() {
  }

  /**
   * Constructor used by {@link org.apache.giraph.examples
   * .SimpleHopsComputation}
   *
   * @param sourceId      the id of the source vertex which wants to
   *                      calculate the hops count
   * @param destinationId the id of the destination vertex between which the
   *                      hops count will be calculated
   */
  public SimpleHopsMessage(long sourceId, long destinationId) {
    this.sourceId = sourceId;
    this.destinationId = destinationId;
  }

  /**
   * Increments the <code>hopsCount</code> by one. Usually when the message
   * arrives at a new node.
   */
  public void incrementHopsCount() {
    this.hopsCount++;
  }

  /**
   * Sets the flag <code>destinationFound</code>. Do this before sending the
   * message back to the source.
   */
  public void setDestinationFound() {
    this.destinationFound = true;
  }

  public boolean isDestinationFound() {
    return this.destinationFound;
  }

  public long getSourceId() {
    return this.sourceId;
  }

  public long getDestinationId() {
    return this.destinationId;
  }

  public int getHopsCount() {
    return this.hopsCount;
  }

  @Override
  public void write(DataOutput dataOutput) throws IOException {
    dataOutput.writeLong(this.sourceId);
    dataOutput.writeLong(this.destinationId);
    dataOutput.writeInt(this.hopsCount);
    dataOutput.writeBoolean(this.destinationFound);
  }

  @Override
  public void readFields(DataInput dataInput) throws IOException {
    this.sourceId = dataInput.readLong();
    this.destinationId = dataInput.readLong();
    this.hopsCount = dataInput.readInt();
    this.destinationFound = dataInput.readBoolean();
  }
}
{% endhighlight %}

You can see, this class looks like any other value-object class, except for implementing the `Writable`-interface. This interface brings with it the two void methods, `write(DataOutput)` and `readFields(DataInput)`. Those are the methods, where you need to serialize or deserialize your class. Because `SimpleHopsMessage` contains only scalar values, the serialization part is easy. You can use the predefined `DataOutput.writeX` and `DataInput.readX` methods. **But make sure, that the order in both methods, `write` and `readFields` remains the same.**

### Internal Vertex-data

`SimpleHopsMessage` was a peace of cake. Now let's have a look at the more complex [SimpleHopsVertexValue](https://github.com/pgrm/giraph/blob/simple-hop-computation/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleHopsVertexValue.java) class. Instead of scalar values, we have now a `Map<Long, Integer>` and a `Set<Map.Entry<Long, Long>>`.

{% highlight java %}
/**
 * The vertices for which the current vertex wants to find out the hop
 * count, together with the number of hops or Integer.MAX_VALUE if the
 * number of hops isn't known yet.
 */
private Map<Long, Integer> verticesWithHopsCount =
        new HashMap<Long, Integer>();

/**
 * A set of already processed request from one source to a target,
 * so that messages won't run in circles.
 */
private Set<Entry<Long, Long>> processedMessages =
        new HashSet<Entry<Long, Long>>();
{% endhighlight %}

Looking at `DataOutput` and `DataInput` there are only ways how to read/write scalar values. So what can we do with complex values? For me, the easiest way of doing it, seemed to be, to serialize  both `Iterable` variables simply element by element. Starting with the count of elements, and than Key followed by Value.

{% highlight java %}
@Override
public void write(DataOutput dataOutput) throws IOException {
  dataOutput.writeInt(this.verticesWithHopsCount.size());

  for (Entry<Long, Integer> entry : this.verticesWithHopsCount.entrySet()) {
    dataOutput.writeLong(entry.getKey());
    dataOutput.writeInt(entry.getValue());
  }

  dataOutput.writeInt(this.processedMessages.size());

  for (Entry<Long, Long> entry : this.processedMessages) {
    dataOutput.writeLong(entry.getKey());
    dataOutput.writeLong(entry.getValue());
  }
}

@Override
public void readFields(DataInput dataInput) throws IOException {
  int size = dataInput.readInt();

  for (int i = 0; i < size; i++) {
    this.verticesWithHopsCount.put(
            dataInput.readLong(), dataInput.readInt());
  }

  size = dataInput.readInt();

  for (int i = 0; i < size; i++) {
    this.processedMessages.add(new SimpleEntry<Long, Long>(
            dataInput.readLong(), dataInput.readLong()));
  }
}
{% endhighlight %}

There is also a different way, how to accomplish something similar, in a more modular fashion, I'll talk about this in a later post. For now, this is sufficient to serialize and deserialize the elements.

Conclusion
---------

Using complex type within Giraph is easy and straight forward. Yet working with large datasets of lists and maps is not very efficient. They can either consume the memory, or slow down performance, when you are serializing and deserializing them.