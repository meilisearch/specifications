- Title: Rename attributesForFaceting
- Start Date: 2021-04-16
- Specification PR: [#35](https://github.com/meilisearch/specifications/pull/35)
- MeiliSearch Tracking-Issues: TBD

# Rename facetsDistribution and attributesForFaceting

## 1. Feature Description and Interaction

### I. Summary
### II. Motivation
As the new engine requires filterable fields to be declared in attributesForFaceting, the name of this field is not as clear as we would like it to be. Since it’s now concerning filter and possible wanted facets on search result by the user.

As we would like to change this name, we are also asking ourselves the question for the facetsDistribution field name which concerns the facets to be output in the search result payload.

### III. Additional Materials
N/A

### IV.Explanation

#### I. attributesForFaceting

`attributesForFaceting` is now used for declaring fields that could be filtered and given as facets.

Since faceting is also used to filter the search result by navigation, the idea is to highlight the filtering aspect in the name of the field.

> Any fields declared in `attributesForFaceting` for filtering can be faceted. Any fields declared in `attributesForFaceting` for faceting can be filtered.

attributesForFaceting is lacking precision because it is now used to authorize filtering and faceting on those fields.

##### Naming ideas
...


#### II. facetsDistribution

> Faceted search, also known as guided navigation or faceted navigation, is a way to add specific, relevant options to your results pages so that when your users search for a product, they can see where in your catalogue they've ended up.

This field is used to display facets and number of matching document for a giving facet. Giving facets allows the user to shape the search result by navigating by clicking on facets in the UI.

##### Naming Ideas

###### facetDistribution

> bidoubiwa (Charlotte) comment
>
> Facet is not a common word. Its mainly used by Algolia and was previously used by elastic search. I feel like its something we can relate to as french people because the etymology of the word is french
one side of something many-sided, especially of a cut gem. The definition makes sense, but is not intuitive for non native English or french speaker.

> react-learner (Tommy) comment
>
> @gmourier I agree that facetsDistribution describes the behavior pretty well; I can't really imagine any scenario besides building a faceted search UI where I would want to use this parameter. However, I also think it's worth noting that with the changes proposed here, facets no longer has a particular meaning within MeiliSearch. As far as the software is concerned, faceting is a specific way of using filters. In other words, knowledge of the term "faceting" is optional to use filters, or even to make a faceted search UI w/ MeiliSearch. So, why not make it optional throughout our software and documentation?
>
>I recognize that "Faceted search" is one of MeiliSearch's desirable features, so I'm definitely not proposing that we eliminate the keyword from the docs—it attracts web traffic and communicates well with the search-focused crowd. However, I think we could background "facets"/"faceting"/"faceted search" by keeping these words off of parameter names, substituting more intuitive names like categoryDistribution or resultDistribution. By doing so, we can reduce friction for people who have never seen the word "facets" before (i.e. less search-focused users: hobbyists, solo devs, IT professionals) without impacting those who are more knowledgeable.
>
> It's an extreme example, but I wouldn't want someone to search "facets meilisearch" on Google and end up on the facetsDistribution reference page instead of our main guide on setting up faceted search. Also, having a single software-level reference to facets when all others have been removed just seems inconsistent to me 🤷‍♂️
>
###### hitsByField

> react-learner (Tommy) comment
>
> This one is really intuitive, since the value of the parameter should be a list of field names. Of course numberOfHitsByField would be more accurate, but that's pretty long. hits could also be replaced by matches or results depending on how we define these terms internally.

###### hitsByCategory

###### result/hit/matchDistribution
> react-leader (Tommy) comment
>
> The nice thing about the word distribution is it instantly conveys that the response will be a number (this is not as clear with hitsByField)

###### documentDistribution
> bidoubiwa (Charlotte) comment
>
> I thought of documentDistribution as we show the distribution of documents between each category. And also, as it clearly informs the users he is going to receive a distribution of documents. So even if it loses in facet context, it at least informs about the return value.

###### filterDistribution
> bidoubiwa (Charlotte) comment
>
> Very common word, but filter distribution implies that we would have that kind of distribution x > 12: 1200 documents.

###### aggregrateDistribution
> bidoubiwa (Charlotte) comment
>
> The word that is now used by ElasticSearch. a whole formed by combining several separate elements.. Which is also technically accurate to describe the distribution. I still feel this is confusing and not intuitive.

### V. Impact on documentation
TBD

### VI. Impact on SDKs
TBD

## 2. Technical Aspects
N/A

## 3. Future possibilities
N/A