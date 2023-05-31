---
layout: default
---


Additional material for the IMC Submission #98 *Connect the Dots: A Graph Model of the TLS Ecosystem and its Internet-wide Application*, providing access to published data and tools.

## Top Centralized IP Addresses

In our paper, we generated a top list of IP Addresses with the highes degrees: [**[csv]**]({{ site.baseurl }}{% link assets/top_1M_centralized_addresses.csv.zst %}):

The file contains four columns: the rank, the IP address, the AS, and the AS Name.
We used [pyasn](https://github.com/hadiasghari/pyasn) to identify the AS during scan time.
In case the AS changed over time, the last two columns are a list.

## Graph-Parsing Pipeline and Example Data

We provide parsing scripts used in our graph-parsing pipeline under [**[Graph Pipeline]**](/graph-pipeline/).
The pipeline explains the steps necessary to reproduce the example graph.

## Paper

**Abstract** 
Independent actors and heterogeneous deployments shape our Internet. With the wide adoption of Transport LayerSecurity (TLS), a whole ecosystem of intertwined entities emerged. An ideal layer for large-scale analyses based on actively collected meta-data. However, models are often necessary to explain this data and to introduce abstractions for further processing. Selecting a proper model to solve a specific problem is challenging. This work proposes a graph-based model of the TLS ecosystem that utilizes the relations among servers, domains, and certificates. Three active Internet-wide Domain Name System (DNS) and TLS measurements were parsed into a single property graph to validate our approach. We found a high centralization and grouped IP Address based on shared responsibilities, revealing far fewer distinct deployments than observed addresses. Combining blocklists with a custom score propagation allowed us to find potentially malicious servers, and we confirmed a high rate of known maliciousness among these servers using external threat intelligence services. This work proposes an approach to help analyze and understand the TLS ecosystem; moreover, it could serve as an addition to the tool-set of security researchers searching for unknown threats on the Internet.

