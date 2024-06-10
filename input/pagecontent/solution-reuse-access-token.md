### Use same access token on all servers.

This approach can work for scenario's where multiple servers are governed by the same organization.

One approach requires the patient id to be aligned on all servers which provides a very strong coupling between servers. This should not be the preferred solution. Referring to single FHIR server for patients might work (other servers use full url references).

For servers that are used by multiple EHR's this might even be impossible unless a specific proxy is provided.
The EHR authorizes access to other resource servers. From a security perspective this is seen as unacceptable if those servers are governed by a different Authorization Server.

#### Approach

The approach taken in this flow is the same as a normal SMART launch. The application detects both servers and determines that access-token-reuse is possible and what scopes are supported by each server. It then sends out a request that contains the total list of scopes.

##### Signal request for multiple servers

The main difference is the recommendation to include multiple `aud` entries in the request (see section on [Obtain authorization code](https://hl7.org/fhir/smart-app-launch/app-launch.html#obtain-authorization-code)). 

##### Passing current patient context

Additionally, the current patient is to be addressed. The current patient in the token-response has to be clarified to where it is located.

This can be approached in multiple ways. As aud is not present in the token-response, the client does not know what FHIR server is used to serve the Patient. To address this the following approach is chosen:

* when `patient` is present, the `patient` field holds the `patient-id` for the current patient in all resource servers.
* when the patient is present in one or multiple servers with a different id, the patient will be included in the fhirContext. This requires a `role` field to be present with the value: `launch-full-url`. This also requires the server to indicate support for `context-style` in `conformance.capabilities`.

An example is presented below:

```json

{
  "need_patient_banner": true,
  "smart_style_url": "https://smart.argo.run/smart-style.json",
  "token_type": "Bearer",
  "scope": "launch/patient patient/Observation.rs patient/Patient.rs",
  "expires_in": 3600,
  "fhirContext" : [
    {   "reference": "http://<ehr-url>/Patient/jdasjdaksjdlasj",
        "role": "launch-full-url"
    },
    {   "reference": "http://<other-url>/Patient/jdasjdaksjdlasj",
        "role": "launch-full-url"
    }
  ],
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuZWVkX3BhdGllbnRfYmFubmVyIjp0cnVlLCJzbWFydF9zdHlsZV91cmwiOiJodHRwczovL3NtYXJ0LmFyZ28ucnVuLy9zbWFydC1zdHlsZS5qc29uIiwicGF0aWVudCI6Ijg3YTMzOWQwLThjYWUtNDE4ZS04OWM3LTg2NTFlNmFhYjNjNiIsInRva2VuX3R5cGUiOiJiZWFyZXIiLCJzY29wZSI6ImxhdW5jaC9wYXRpZW50IHBhdGllbnQvT2JzZXJ2YXRpb24ucnMgcGF0aWVudC9QYXRpZW50LnJzIiwiY2xpZW50X2lkIjoiZGVtb19hcHBfd2hhdGV2ZXIiLCJleHBpcmVzX2luIjozNjAwLCJpYXQiOjE2MzM1MzIwMTQsImV4cCI6MTYzMzUzNTYxNH0.PzNw23IZGtBfgpBtbIczthV2hGwanG_eyvthVS8mrG4",
  
}

```

##### Detect support for access-token-reuse

The client needs to detect that the other server can be addressed using the same access-token. According to the current specification this is achieved by listing the other server in  `associated-endpoints` but this will provide no information on the type of server. It would be better to move this in the brand-bundle.

**TODO:** Investigate this further.

The fact that access-token could be reused could be derived from the fact that both hold the same `authorization_endpoint` and `token-endpoint`. It is best to also list it in the `capabilities` array, e.g. using:

* `auth-multi-aud-token`: support for access tokens for more than one resources server (aud).

#### Flow

The typical launch flow is presented below.

<figure>
  {% include solution-reuse-access-token.svg %}
  <figcaption><b>Figure: Reuse access token.</b></figcaption>
  <p></p>
</figure>

#### Required changes

[RFC6749](https://www.rfc-editor.org/rfc/rfc6749.html)

Changes required in the SMART application launch specification.

**CHANGE**: Allow multiple aud's in the [Obtain authorization code](https://hl7.org/fhir/smart-app-launch/app-launch.html#obtain-authorization-code) call.

**CHANGE**: Define the role `launch-full-url` as a full url alternative of the launch patient and encounter, alternatively allow entries with launch for full urls in [fhirContext](https://hl7.org/fhir/smart-app-launch/scopes-and-launch-context.html#fhircontext-exp).
