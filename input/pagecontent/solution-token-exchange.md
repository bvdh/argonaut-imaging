### Use token exchange to obtain an access token for the other server

Using token-exchange (see [RFC8693](https://www.rfc-editor.org/rfc/rfc8693.html) an access-token for the other server is retrieved. This will work for vendor apps with user scopes. Patient scopes require the patient to be passed. A possible solution for this would be to include the patient-id and source in the open-id token.

#### Approach

The approach taken in this flow is the same as a normal SMART launch. The application detects both servers and determines that access-token-reuse is possibe and what scopes are supported by each server. It then sends out a request that contains the total list of scopes.

This approach does not include a new step in which the user is requested to approve the application having access to the data of the other server. This makes it a solution that might work for vendor apps but not for patient facing apps.

##### Signalling the supported open-id providers

In order for the App to known what open-id tokens are accepted.

##### Determine server supports token-exchange with open-id token.

Signalling a FHIR server supports token exchange with an open-id or access-token is signalled in the following way:

1. `conformance.grant_types_supported` will hold `urn:ietf:params:oauth:grant-type:token-exchange` signalling support for token exchange.
2. For systems supporting open-id based token-exchange:
   1. `conformance.scopes_supported` will hold `open-id` signalling that `open-id` is supported.
   2. `conformance.capabilities` will hold `token-exchange-openid` 
3. For systems supporting access-token based token-exchange:
   1. `conformance.capabilities` will hold `token-exchange-accesstoken` 

##### Determine open-id/access-tokens that are accepted

This could be done in several ways. It could be pre-configured and determined out of bounds. In which case additional specificat is required. This is an option as, as is stated above, token-exchange is most likely used for vendor-facing applications.

Alternatively, the authorization server could be indicated in the `conformance.authorization_endpoint`. This will work if the server only accepts tokens from one such server (the field is 0..1). In the case the resource server accepts both normal OAuth flows as well as token exchange, this will not suffice. It is also not completely correct as it points to the authorization not token-endpoint and the open-id token is issued as part of the retrieve access token flow.

Open-id conncet has specified a discovery mechanism in [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html). This relies on [RFC7033-webfinger](https://www.rfc-editor.org/rfc/rfc7033.html#section-3.1). This is similar to SMART's discovery mechanism and providers information on the OpenId provider, not about the supported open-id tokens.

We could add a entry to conformance listing the supported access-token and open-id provider discovery endpoints or add them to the `associated_endpoint` list. The latter approach is used. The capability used to indicate token-exchange support will be the same as is used in the capability list: `token-exchange-openid` and `token-exchange-openid`.

##### Passing current patient context

When using token-exchange using access-tokens, the patient id can be retrieved using introspection. That would not work for open-id based token-exchange. Passing access-tokens is a security vulnerability should be avoided if possible.

There are different ways to address this issue. The first one is to limit token-exchange to scenerio's where only a `system` and `user` specific scopes are supported; `patient` specific scopes are refused or translated into `system` and `user` specific scopes.

Alternatively, the current patient could be encoded in the open-id token, although this is contrary to what the token represents.

For now we assume no `patient` scoped requests are supported or there is an out of bound mechanisms supporting this (e.g. embed it in the open-id token). The different Authorization Systems could also align on some state related to the current user, passing this state in the token-response. This could be made available when performing introspection on the open-id token.

If it is supported, the same rules apply as in reuse access tokens:

* when `patient` is present, the `patient` field holds the `patient-id` for the current patient in all resource servers.
* when the patient is present in one or multiple servers with a different id, the patient will be included in the fhirContext. This requires a `role` field to be present with the value: `launch-full-url`.

##### Credentials for introspection

The different authorization systems are connected by setting up SMART Back-end services authorizations that allow introspection of access and open id tokens.

#### Flow

The typical launch flow is presented below.

<figure>
  {% include solution-token-exchange.svg %}
  <figcaption><b>Figure: Token-exchange.</b></figcaption>
  <p></p>
</figure>

#### Required changes

Changes requried in the SMART application launch specification.

**CHANGE**: Add `grant_type` and `subject_type` and `subject_token_type`

**CHANGE**: Add `token-exchange-openid` and `token-exchange-accesstoken` capabilities.

**CHANGE**: Define the role `launch-full-url` as a full url alternative of the launch patient and encounter, alternatively allow entries with launch for full urls in [fhirContext](https://hl7.org/fhir/smart-app-launch/scopes-and-launch-context.html#fhircontext-exp).
