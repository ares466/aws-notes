# Amazon DataZone

- [Overview](#overview)
- [Example Workflow](#example-workflow)
- [Resources](#resources)

## Overview

Amazon DataZone introduces several new components:
- *Business data catalog*: Makes metadata available with business context so anyone in the organization can find data.
- *Data projects*: Construct that enables you to define business use case-based groupings of people, data, and analytics tools. Projects provide users a space where members of the  projects are able to collaborateand share a common set of permissions to adata and tools.
- *Publish workflow*: Enables data producers to easily publish their data to the catalog.
- *Subscribe workflow*: Enables data consumers to subscribe to, or access, data that has been published to the catalog.
- *Governance and access control*: The publish and subscribe workflow have governance mechanisms to enforce access controls: Who is allowed to publish data to the catalog?, Who is responsible for ownership of the data? Who can subscribe to the data?
- *Data Portal*: A web-based UI in which DataZone users can experience all the capabilities of DataZone.

<img src="./images/AmazonDataZone.png" width="700px">

## Example Workflow

Consider the following example: *The marketing team needs access to sales data for a marketing project. With this data, the marketing team must run analytics for an upcoming promotion.*  

<img src="./images/SampleWorkflow.png" width="700px">

This organization will need to perform the following tasks in order to enable the marketing team:

1. Create DataZone Domain

**PUBLISHING WORKFLOW**

2. Create *SalesProducerProject*
3. Create `catalog_sales` Glue table
4. Publish `catalog_sales` table to DataZone

**SUBSCRIPTION WORKFLOW**

5. Create *MarketingConsumerProject*
6. Subscribe to `catalog_sales` table
7. Query the data using Athena.

## Resources

- [Amazon DataZone](https://aws.amazon.com/datazone/) 
- [YouTube - How to Get Started with Amazon DataZone](https://www.youtube.com/watch?v=9iAvHR1_CiA)
