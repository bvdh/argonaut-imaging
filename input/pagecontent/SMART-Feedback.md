## SMART configuration

Over the last years many new entries have been proposed by SMART and others. May a more flexible extension mechanism consisting of tag-value would be more appropriate.
The tag could be modelled as a extensible value set.

## Bundle type

Current type is `collection` which makes it impossible for the link in smart-configuration to consist of a search query.

## User Brand Endpoint

Slicing on Endpoint.payloadType is missing. The current expression requies all codings to be http://terminology.hl7.org/CodeSystem/endpoint-payload-type#none. This makes it impossible to also add other codings, is this the intention? If so, why is the cardinality 1..*?

## User Brand Organization

The endpoints are restricted to UserBrandEndpoint making it impossible to add other options. I do not think this is the preferred solution.