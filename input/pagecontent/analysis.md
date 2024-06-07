Solution directions considered:

{% include solution-reuse-access-token.md %}

{% include solution-token-exchange.md %}

{% include solution-dual-smart-launch.md %}

### Authorization proxy

Proxy server that takes the authentication request and authorizes with multiple Authorization Servers. This a new field for which there are no current specifications.

It also requires all parties to trust the proxy server. It would allow the authorization flow to follow the normal OAuth2 flow from the client perspective.