- Title: Telemetry Policies
- Start Date: 2021-04-16
- Specification PR: [#34](https://github.com/meilisearch/specifications/pull/34)
- MeiliSearch Tracking-Issues: [transplant/#181](https://github.com/meilisearch/transplant/issues/181)

# Telemetry Policies

## 1. Feature Description and Interaction

### I. Summary

Telemetry concerns the possibility of depositing markers within a system in order to make segments of it. Quantitative data as data metrics are used to discover and verify user behavior.

We would like to have more telemetry to better understand our users in order to improve their usage and feel with MeiliSearch.

We also use Sentry to trace the errors that occur when using MeiliSearch.

### II. Motivation

We need to make informed decisions based on data instead of instincts and gut feel. Lack of data does not say anything about how correct or wrong are our intuitions. Lack of data means we did not make testable hypotheses. We will now support our decisions with data.

> 🔒 Note that we strictly don't want to collect private information by default (ip address, email, website url, etc..).

We want to simplify the disabling of telemetry and better inform users about what we want to track.

### III. Additional Materials
N/A

### IV.Explanation

Following this issue and with the beginning of a product Team we asked ourselves what we wanted to do about telemetry. https://github.com/meilisearch/transplant/issues/141.

We will keep telemetry enabled by default while leaving the option for users to disable it. See Impact on Documentation part.

> Some new features will have telemetry data points to check the assumptions and measure objectives achievements.

Each new feature specification should list data points to be collected and measured. Obviously the specification must explain why these data points are necessary for relation to the feature.

We will simplify the deactivation of all our telemetries in one action. To do that, we will remove the Sentry specific flag `MEILI_NO_SENTRY` to merge its use into the `MEILI_NO_ANALYTICS` flag. This means that its activation and deactivation will be shared with analytics within the `MEILI_NO_ANALYTICS` flag. By default, Sentry metrics will stay activated.

#### Current stack for analytics

We are currently using Amplitude as a quantitative analytic tool to measure usage as a big picture.

E.g.
- Cohort: Servers that are running for more than seven days with the old version of MS (< v0.10.1)
- Cohort: Servers that are running for more than seven days with the new version of MS (> v0.10.1)
- Cohort: Servers that are running for more than 30 days with the old version of MS (< v0.10.1)
- Cohort: Servers that are running for more than 30 days with the new version of MS (> v0.10.1)
- Graph: The total of every telemetry messages received
- Graph: The total of every unique id seen
- Graph: The number of servers that are running at least since 7 days
- Graph: The number of servers that are running at least since 30 days
- User Composition: Device composition
- User Composition: Version composition
- Indexes size distribution for more than seven days (<200k not taken into account)
- Indexes size distribution for the others

#### Future Metrics

##### Number of indexes per DB
Discovery metric for federated search feature.

##### Number of documents per indexes
Discovery metric to know the average number of documents are stored in an index.

##### Geographical server distribution, CPU server distribution, RAM server distribution, Disk server distribution
Discovery metrics to choose more relevant data centers over the world for the SaaS cloud platform. Will also be used in order to perform price studies analysis.

For the Geographical Server Distribution we need to make a call to a third-party like http://ip-api.com/json/113.14.168.85 to get `Country`, `City` and `Provider` at MeiliSearch launch.

Staying transparent is important to us, which is why the metrics will be explained in details on a change log, blog post and a dedicated documentation page.

For the CPU, RAM and DISK metrics, we can use an internal system stat method to get this information.

##### SDK distribution
Discovery metric to check adoption of our new sdks and the use of existing ones

##### Feature metrics
E.g. We want to know more about feature usage like how many times the `ranking_rules` order are changed, which is the top most used `ranking_rules` in first position etc.. What is the average `response body` size, what is the average number of `filter` used for the search endpoint.

❗️ Asking for new data points should be specified and explain why these data points will be used for in the feature specification.

### V. Impact on documentation

Create a dedicated page aimed to explain why and how we collect anonymous data while being exhaustive on the metrics we collect. Related to https://github.com/meilisearch/documentation/issues/908.
Remove the [Sentry part](https://docs.meilisearch.com/reference/features/configuration.html#disable-sentry) and mention it on the future dedicated page and in the [Analytics part](https://docs.meilisearch.com/reference/features/configuration.html#analytics).

### VI. Impact on SDKs
N/A

### VII. Impact on CLI

Currently, MeiliSearch only says if `Anonymous telemetry` is enabled or not on the launch message. It should also provide a message explaining in a few words that we are collecting anonymized MeiliSearch behavior metrics to enhance the product for future releases. Thus, displaying a link to the analytics documentation page.

Message to display on the CLI at launch if analytics are enabled:
> "Thank you for using MeiliSearch! We collect anonymized analytics to improve our product and your experience. To learn more, including how to turn off analytics, visit our dedicated documentation page: [Link of the new dedicated page]()."

## 2. Technical Aspects

### I. Abstract

⚠ Telemetry should not impact performances for our users.

### II. Technical Details

#### Amplitude HTTP V2 Endpoint

Currently MeiliSearch uses a deprecated endpoint to send data to Amplitude. A new endpoint exists and could better suit our needs. [See here](https://developers.amplitude.com/docs/http-api-v2).

We want to use the `http-api-v2` endpoint.

#### Logging

Errors when sending metrics to Amplitude or Sentry should be silent.

## 3. Future possibilities

### Segment to collect data

Why we consider using Segment to collect and send data ?

- Segment will represent our unique source of collecting data. It permits to change on the fly and fill data to new analytics products without loosing data.
- It seems to offer smarter mechanisms for collect. Thus, having a very low impact on performance of the system on which it collects data. [See the rust docs here](https://segment.com/docs/connections/sources/catalog/libraries/server/rust/)

## 4. Planned Changes

### 0.21 Release

- Use `http-api-v2` endpoint to send data to Amplitude.
- Display a CLI message at Meilisearch start explaining why we collect analytics and how to deactivate it.
- Merge Sentry activation/deactivation in Analytics activation/deactivation flag.
- Make errors when sending analytics to Amplitude and Sentry silents.
- Make a dedicated documentation page to explain why we collect analytics, what we are collecting, how to disable it. This dedicated documentation page will be linked into the new CLI message.

### 0.22 Release

- Branch telemetry at feature level.