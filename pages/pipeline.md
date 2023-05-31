---
layout: page
title: Graph-Pipeline
permalink: /graph-pipeline/
--- 

# Graph-Parsing Pipeline and Example Data

To enable reproducable results and help in understanding our approach we have open-sourced the full parsing pipeline that can create the ITEgraph from our paper.
Additionally, the repository contains an example graph created from the [Tranco](https://tranco-list.eu/) top 1M list that can be directly imported and explored via [Neo4J](https://neo4j.com/).
The code and data can be found [[here]](https://github.com/imc2023-98/graph-pipeline) or directly via Git:

{% highlight bash %}
git clone git@github.com:imc2023-98/graph-pipeline.git
cd graph-pipeline
git lfs pull
{% endhighlight %}

Git-LFS is only used for the example graph.
For simplicity, we excluded the full IPv4 address space scan (performed with [ZMap](https://zmap.io/)) and the scan for an open port 443 on IPv6 Addresses with [zmapv6](https://github.com/topics/zmapv6).
In the following we explain the steps necessary to reproduce the `example-data/` from the repository.

## 1. The Domain Name Input

The initial input for our ITEgraph was a domain name list. For this example, we used the [Tranco](https://tranco-list.eu/) top 1M domains:

{% highlight bash %}
curl https://tranco-list.eu/download/LYZG4/full | cut -d , -f 2 | head -n 1000000 > example-data/input.txt
{% endhighlight %}

## 2. DNS Resolutions

We have used a local [unbound](https://www.nlnetlabs.nl/projects/unbound/about/) server and [massdns](https://github.com/blechschmidt/massdns) for our large-scale DNS resolutions.
Any software would work that produces a CSV file like the following:

    172.217.16.206,google.com
    2a00:1450:4001:806::200e,google.com

We started with a list of domains names (`example-data/input.txt`), resolved them via massdns, and parsed the output into the proper format:

{% highlight bash %}
massdns  -o J -t AAAA -t A --filter NOERROR -r ./example-data/resolvers.txt example-data/input.txt  | ./parse_massdns.py --output example-data/dns
{% endhighlight %}

This creates two files each for the IPv4 and IPv6 DNS resolutions.

## 3. The TLS Scan

Our TLS measruements were performed with the [TUM goscanner](https://github.com/tumi8/goscanner.git).
The scanner needs a single input file:

{% highlight bash %}
cat example-data/dns/*.ipdomain | shuf > example-data/goscanner-input.csv
{% endhighlight %}

Then, we can start with the actual TLS measurements:

{% highlight bash %}
ulimit -n 1024000
goscanner -C example-data/goscanner.conf --input-file example-data/goscanner-input.csv --output example-data/tls
{% endhighlight %}


## 4. Combining it all as ITEgraph

We created the ITEgraph with [Apache Spark](https://spark.apache.org/).
Spark can be tricky to set up; hence, we provide docker scripts that set up the environment and parse the example data.
Have a look at the `docker-compose.yml` and the parsing script for further details.
The script extracts the scan dates from the directory name of the input.
Note the necessary cache directory that can be deleted after the run.

{% highlight bash %}
docker-compose up --exit-code-from spark && docker-compose rm -sf
{% endhighlight %}

## 5. Explore and Analyze the ITEgraph

Our parsing pipeline produces multiple files for each type of edge and node that can be analyzed with other tools.
For example, the CSV output under `example-data/ITEgraph` can be directly imported into Neo4J.

{% highlight bash %}
./import_neo4j.sh example-data/ITEgraph
{% endhighlight %}


We experienced that Neo4J provides a convenient Interface to manually explore the graph; however, for more advanced analyses we recommend tools like Apache Spark with the [GraphFrames](https://graphframes.github.io/graphframes/) library.

The output of the Neo4J schema function:

![Schema of the Example-Graph](/assets/example_schema.svg){:style="display:block; margin-left:auto; margin-right:auto"}

Note that there were IP Addresses embedded as Altertnative Name in some certificates. These were always self-signed certificates.
However, we did not consider such cases in the paper.
