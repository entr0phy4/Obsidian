docs: https://docs.netbird.io/selfhosted/selfhosted-guide

```
NETBIRD_AUTH_OIDC_CONFIGURATION_ENDPOINT=`http://localhost:8080/realms/netbird/.well-known/openid-configuration`.
NETBIRD_USE_AUTH0=false

NETBIRD_AUTH_CLIENT_ID=`netbird-client`
NETBIRD_AUTH_SUPPORTED_SCOPES="openid profile email offline_access api"
NETBIRD_AUTH_AUDIENCE=`netbird-client`

NETBIRD_AUTH_DEVICE_AUTH_CLIENT_ID=`netbird-client`
NETBIRD_AUTH_DEVICE_AUTH_AUDIENCE=`netbird-client`

NETBIRD_MGMT_IDP="keycloak"
NETBIRD_IDP_MGMT_CLIENT_ID="netbird-backend"
NETBIRD_IDP_MGMT_CLIENT_SECRET="wAxJpimnjf3x0RjobQeM1AM7wPalB5lY"
NETBIRD_IDP_MGMT_EXTRA_ADMIN_ENDPOINT="http://localhost:8080/admin/realms/netbird"
```

[[Keycloak]]