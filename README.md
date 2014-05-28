# Giraph Debugger
## Overview:
TODO(semih): Write

## Synopsis

### Get Protocol Buffers
If your system doesn't have `protoc` command, you need to install [Protocol Buffers](https://code.google.com/p/protobuf/):

    wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
    tar xfz protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0
    ./configure
    make
    make check
    make install

### Get Giraph Trunk
    git clone https://github.com/apache/giraph.git -b trunk
    cd giraph
    mvn -DskipTests --projects .,giraph-core install

### Get Graft under Giraph, Build and Install It
    git clone https://github.com/semihsalihoglu/distributed_graph_debugger.git giraph-debugger
    cd giraph-debugger
    mvn -DskipTests compile
    
    PATH=$PWD:$PATH  # Add current directory to PATH, so we can easily run giraph-debug later

### Download a Sample Graph
    curl -L http://ece.northwestern.edu/~aching/shortestPathsInputGraph.tar.gz | tar xfz -
    hadoop fs -put shortestPathsInputGraph shortestPathsInputGraph

### Launch Giraph's Shortest Path Example
    cd ../giraph-examples
    mvn -DskipTests compile
    hadoop jar \
        target/giraph-examples-*.jar org.apache.giraph.GiraphRunner \
        org.apache.giraph.examples.SimpleShortestPathsComputation \
        -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat \
        -vip shortestPathsInputGraph \
        -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat \
        -op shortestPathsOutputGraph.$RANDOM \
        -w 1 \
        -ca giraph.SplitMasterWorker=false \
        #

### Now, Launch It with Debugging
You can launch any Giraph program with debugging support by simply replacing the first two words (`hadoop jar`) of the command:

    giraph-debug \
        target/giraph-examples-*.jar org.apache.giraph.GiraphRunner \
        org.apache.giraph.examples.SimpleShortestPathsComputation \
        -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat \
        -vip shortestPathsInputGraph \
        -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat \
        -op shortestPathsOutputGraph.$RANDOM \
        -w 1 \
        -ca giraph.SplitMasterWorker=false \
        #

Find the job identifier from the output, e.g., `job_201405221715_0005` and copy it for later.

You can optionally specify the supersteps and vertex IDs to debug:

    giraph-debug -S 0 -S 1 -S 2 -V 1 -V 2 -V 3 \
        target/giraph-examples-*.jar org.apache.giraph.GiraphRunner \
        # ... rest are same as above

### Launch Debugger GUI
Launch the debugger GUI with the following command:

    giraph-debug gui

Then open <http://localhost:8000> from your web browser, and paste the job ID to browse it after the job has finished.

If necessary, you can specify a different port number when you launch the GUI.

    giraph-debug gui 12345

### Or, Stay on the Command-line to Debug

You can access all information that has been recorded by the debugging Giraph job using the following commands.

#### List Recorded Traces

    giraph-debug list
    giraph-debug list job_201405221715_0005

#### Dump a Trace

    giraph-debug dump job_201405221715_0005 0 6

#### Generate JUnit Test Case Code from a Trace

    giraph-debug mktest        job_201405221715_0005 0 6 Test_job_201405221715_0005_S0_V6
    giraph-debug mktest-master job_201405221715_0005 0   TestMaster_job_201405221715_0005_S0

