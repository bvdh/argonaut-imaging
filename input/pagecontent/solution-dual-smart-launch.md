### Login on multiple servers

The OAuth2 flow requires a redirect for login. Multiple redirects in a row will cause issues for the application as it becomes difficult to maintain state.

This is required though when notifying the user of the access it will provide. For vendor apps this is not always required and might be skipped.

It is a scenario that is required when the user has to provide permission to access the content in secure manner.

#### Approach

The approach taken in this flow is the same as a normal SMART launch, but couples several of these back-to-back. The advantage of the approach is that it does not compromise the SMART on FHIR approach and requires little to no additional specification. The disadvantage is that it requires multiple redirects with state maintained between them.

##### Passing current patient context

The patient context of the first launch is passed when performing the first SMART launch, triggered by the launch token. This information needs to be passed to the second launch. This requires alignment between the different authorization systems. In a way this is similar to the token exchange approach where a similar alignment is required.

Not this is only required when patient specific scopes are used.

One approach to address this would be use the context received from the first launch to create a launch token for the second launch; standardizing the structure of the launch token or providing an endpoint to provide it.

#### Flow

The typical launch flow is presented below.

<figure>
  {% include solution-multiple-smart-launch.svg %}
  <figcaption><b>Figure: Multiple SMART launch.</b></figcaption>
  <p></p>
</figure>

#### Required changes

Changes requried in the SMART application launch specification.

**CHANGE**: Add `grant_type` and `subject_type` and `subject_token_type`

**CHANGE**: Add `token-exchange-openid` and `token-exchange-accesstoken` capabilities.

**CHANGE**: Define the role `launch-full-url` as a full url alternative of the launch patient and encounter, alternatively allow entries with launch for full urls in [fhirContext](https://hl7.org/fhir/smart-app-launch/scopes-and-launch-context.html#fhircontext-exp).
