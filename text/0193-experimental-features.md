# Experimental features

## 1. Summary

Experimental features are features that are not covered by [Meilisearch's stability guarantee](https://github.com/meilisearch/engine-team/blob/main/resources/versioning-policy.md). They require explicit opt-in to be usable and may break back compatibility or be removed between minor and patch versions of Meilisearch.

## 2. Motivation

See [Motivation](https://github.com/meilisearch/engine-team/blob/main/resources/experimental-features.md#motivation) in the experimental feature process.

## 3. Functional Specification

Some experimental features can be enabled when starting the Meilisearch binary via a command line option, a configuration option or an environment variable. These experimental features are the [instance experimental features](#31-instance-experimental-features).

Other experimental features can be enabled at runtime by using the [`/experimental-features` route](./0194-experimental-feature-api.md). They are the [runtime experimental features](#32-runtime-experimental-features).

## 3.1. Instance experimental features

To enable instance experimental features, pass their associated command line flag as an option to Meilisearch. For each such command line flag, there also exists a configuration option and an environment variable.

|Feature name|Command line flag|Description|Missing for stabilization|Expected stabilization date/version|Product discussion|
|------------|-----------------|-----------|-------------------------|-----------------------------------|------------------|
|Prometheus Metrics|`--experimental-enable-metrics`|The `/metrics` endpoint exposes metrics to be scraped by a Prometheus collector at regular intervals and stored for analysis.|We have yet to determine which metrics we want to expose and how|TBC|<https://github.com/meilisearch/product/discussions/625>|
|Reduce Indexing Memory Usage|`--experimental-reduce-indexing-memory-usage`|Trades-off indexing speed with a lower RAM footprint.|We have yet to determine if a lot impacts performance and whether the RAM usage reduction is significant enough.|TBC|<https://github.com/meilisearch/product/discussions/652>|


## 3.2. Runtime experimental features

<table>
    <thead>
        <tr>
            <th>Feature name</th>
  			<th>How to enable</th>
  			<th>Description</th>
  			<th>Missing for stabilization</th>
  			<th>Expected stabilization date/version</th>
            <th>Product discussion</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><a id="user-content-vector-store" class="anchor" aria-hidden="true" href="#vector-store">vector store</a></td>
            <td><!-- The newline below is required for the markdown to be rendered by GitHub -->

```
curl \
  -X PATCH 'http://localhost:7700/experimental-features/' \
  -H 'Content-Type: application/json'  \
--data-binary '{
    "vectorStore": true
  }'
```

 </td><!-- The newline above + starting from column 0 are required for the markdown to be rendered by GitHub -->
  			<td>Enables storing and searching by using semantic vectors</td>
  			<td>Confidence in the speed of the indexation, search of the vectors, and API surface</td>
  			<td>v1.4</td>
            <td>

<https://github.com/meilisearch/product/discussions/677>

</td>
        </tr>
        <tr>
            <td><a id="user-content-score-details" class="anchor" aria-hidden="true" href="#score-details">score details</a></td>
            <td><!-- The newline below is required for the markdown to be rendered by GitHub -->

```
curl \
  -X PATCH 'http://localhost:7700/experimental-features/' \
  -H 'Content-Type: application/json'  \
--data-binary '{
    "scoreDetails": true
  }'
```

 </td><!-- The newline above + starting from column 0 are required for the markdown to be rendered by GitHub -->
  			<td>Enables computing detailed scores</td>
  			<td>Confidence on the computed details names</td>
  			<td>v1.4</td>
            <td>

<https://github.com/meilisearch/product/discussions/674>

</td>
        </tr>
    </tbody>
</table>



## 4. Technical Details

N/A

## 5. Future Possibilities

N/A