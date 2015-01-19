---
title: "First steps with Apache-Giraph"
tags: 
  - First steps
  - Apache-Giraph
  - graph-processing
---

I used [Apache Giraph](http://giraph.apache.org/) during a project for my studies. There is a great [official Quick Start-Tutorial](http://giraph.apache.org/quick_start.html) on how to set up Giraph on top of Apache Hadoop 0.20.203.0-RC1 and run your first example application. When you follow it, you will have a running environment where you can test your code. Still, you won't have any idea, how to write it (at least I didn't). 

A friend of mine pointed me to her blog posts about Giraph:

- [Run Example in Giraph: Shortest Paths](http://marsty5.com/2013/04/29/run-example-in-giraph-shortest-paths/)
- [Run example in Giraph: PageRank](http://marsty5.com/2013/05/29/run-example-in-giraph-pagerank/)

This gave me a clearer picture. I understood better, what is happening and how to make new things. *(Think as a vertex, you are the vertex.)* But there was still a clear explanation of the code missing. Ok, Apache Giraph is an open source project, and there is a [giraph-examples folder](https://github.com/apache/giraph/tree/release-1.0/giraph-examples/src/main/java/org/apache/giraph/examples) in the repository, so let's just have a look at it.

Making sense of the code for [SimpleShortestPathVertex](https://github.com/apache/giraph/blob/release-1.0/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleShortestPathsVertex.java) wasn't too difficult. It is only few lines, which are relevant and if we remove the logging and add some comments it will be actually easy to grasp it:

{% highlight java %}
public void compute(Iterable<DoubleWritable> messages) {
  if (getSuperstep() == 0) {
    // initially, no node is reachable
    setValue(new DoubleWritable(Double.MAX_VALUE));
  }
  
  double minDist = isSource() ? 0d : Double.MAX_VALUE;
  // check the incoming messages 
  // and collect the minimal distance
  // (for the source this will be always 0)
  for (DoubleWritable message : messages) {
    minDist = Math.min(minDist, message.get());
  }
  
  // check if the new minimal distance is smaller 
  // than the original one (Double.MAX_VALUE initially)
  // this will be in the beginning only true for 
  // the source vector since its distance is always 0
  if (minDist < getValue().get()) {
    setValue(new DoubleWritable(minDist));
    for (Edge<LongWritable, FloatWritable> edge : getEdges()) {
      // send a message to all neighbours with 
      // the current distance from the source vector
      // + the distance to the neighbour
      double distance = minDist + edge.getValue().get();
      sendMessage(edge.getTargetVertexId(), new DoubleWritable(distance));
    }
  }
  
  // task is finished, until new messages will arrive
  // in this case the vertex will be woken up again
  voteToHalt();
}
{% endhighlight %}

Now the problem with this example is, that it's too easy. All messages and the internal state consist of only one `Double`-Value. In reality, however, this won't be enough, and I just could not find anything that would describe the next steps. So I had to experiment and eventually wrote my own simple algorithm to learn how to write code for Giraph.

In the upcoming days I'll publish more blog posts about:

- [Implementing a simple hop-count (or modified shortest-path) algorithm in Giraph](http://peter.grman.at/implementing-a-simple-hop-count-algorithm-in-apache-giraph/) ([source code](https://github.com/pgrm/giraph/blob/simple-hop-computation/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleHopsComputation.java))
- Implementing [Ja-be-Ja](https://www.sics.se/~amir/files/download/papers/jabeja.pdf) algorithm in Giraph ([source code](https://github.com/pgrm/giraph/tree/jabeja/giraph-examples/src/main/java/org/apache/giraph/examples/jabeja))
- And why not to use Giraph for edge-centric algorithms