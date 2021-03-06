---
title: Processing Trucking IoT Data with Apache Storm
tutorial-id: 800
platform: hdp-2.6.0
tags: [storm, trucking, iot, kafka]
---

# Processing Trucking IoT Data with Apache Storm

## Introduction

This tutorial will cover the core concepts of Storm and the role it plays in an environment where real-time, low-latency and distributed data processing is important.

We will build a Storm topology from the ground up and demonstrate a full data pipeline, from Internet of Things (IoT) data ingestion from the edge, to data processing with Storm, to persisting this data and viewing it in a visualization web application.


## Prerequisites

-   [HDP 2.6 Sandbox Installed](https://hortonworks.com/downloads/#sandbox) (coming soon)


## Outline

-   **Trucking IoT Use Case** - Discuss a real-world use case and understand the role Storm plays within in.
-   **Running the Demo** - Walk through the demo and first gain an understanding of the data pipeline.
-   **Building a Storm Topology** - Dive into Storm internals and build a topology from scratch.
-   **Deploying the Topology** - Package the Storm code and deploy onto your cluster.
