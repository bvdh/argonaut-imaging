The main discovery flow is described in the diagram below.

<figure>
  {% include brand-based-discovery.svg %}
  <figcaption><b>Figure: Brand based endpoint discovery.</b></figcaption>
  <p></p>
</figure>

This allows discovering of the endpoints. It does not yet provide information on the security related information that is required for a client to connect. That information can be retrieved using the `<fhir>/.wellknown/smart-configuration` endpoint.