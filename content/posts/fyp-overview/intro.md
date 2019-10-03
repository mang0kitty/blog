---
title: "Automating the parallel alignment of DNA using Kubernetes jobs"
date: 2019-08-19T20:37:20+01:00
draft: true
author: Aideen
description: Overview of my final year project to reduce technical overhead and time for geneticists to perform alignment jobs.
cover:
tags:
  - kubernetes
  - automation
  - monitoring
  - benchmarking
  - bwbble
---

After much deliberation, I’ve settled on a final year project and have planned and specked it out with the help of Azure DevOps. There was an opportunity to implement ESNI (Encrypted Server Name Indicator) into NGINX as part of an IETF draft. I’d love to hack away at this and hope to do so in my free time, however I’ve decided to take a project to deploy “BWBBLE”, a read-alignment program, to Kubernetes and extend the proposal to involve automating the parallel alignment of DNA to reduce the time taken for geneticists to perform a BWBBLE alignment job. This will be done using modern job orchestration tools and patterns paired with a simple and easy to use interface for the geneticists to interact with.

For background into BWBBLE and why I though this a problem worth solving, BWBBLE is a read-alignment program that maps short DNA sequences (short reads) to a multi-reference genome. The bottleneck of most DNA analysis pipelines is this read-alignment phase. More-over, BWBBLE is up to 100 times slower than other read aligners like SOAP2 or BWA, due to vast amount more data it processes using a multi-reference genome. BWBBLE is the only read-alignment tool that can map a newly sequenced genome to a large collection of genomes i.e a multi-reference genome. This avoids the lower accuracy and inherent biases that result when using a single reference genome. I thought helping to speedup this program would be an awesome problem to solve but more so the fact that I can really uncover how to build a reliable, scalable (to the optimum scale for this embarrassingly parallel problem!) and efficient system. Right now, the program operator needs to manually split up the reads file to be shared. I want to automate this and have a service that geneticists can use.

The project presents challenging questions, like what are the security implications of using Kubernetes and how can I identify the BWBBLE workload stages and bottlenecks. I already have some ideas on how to evaluate different runtime configurations for performance. Reading Brendan Gregg’s System Performance has really got me excited about the benchmarking and monitoring I will get to do. For example, to simplify the operation and maintenance of the service, a monitoring system built upon Prometheus and Grafana will be integrated – enabling real-time visualization of the cluster’s performance and (potentially) the process made in processing individual jobs.

I will look at data storage in the cloud, maybe using Blob Storage/a combination of shared storage (used to provide the reference genome and “reads” files, as well as acting as a destination for results) and build a control plane service for BWBBLE on Kubernetes which will be responsible for sharding “reads” files and initiating jobs will be implemented.

Apart form the technical aspects of this project, exploring genetics and bio-informatics as field within themselves has been absolutely fascinating. The BWBBLE program uses BWT as a compression algorithm to create and indexed data structure of the multi-reference genome to speed up read alignment. Google has a great playlist on other compression algorithms and how they really impact our experience of the internet.

“The Gene: An Intimate History” by Siddhartha Mukherjee is a beautiful and haunting book tracing the importance of hereditary and how it shaped some of our most horrific historical events. In addition to looking back at the past, Mukherjee looks at the present and presents his vision into the future of genome science. “A Crack in Creation” by the Co-creator of CRISPR is another great book that tugs at the question of what we will do now that we know the human code and have technologies that could change what it means to be human itself.

I’m very excited by this project and right now I’m on my first real sprint week and have run BWBBLE on my local machine and on Docker too. The next step is to run BWBBLE in parallel which will involve a lot of research and possibly updating the Helm chart to enable running parallel jobs on K8s.

Thanks for reading!
