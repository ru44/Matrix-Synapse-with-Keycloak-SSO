# Matrix Synapse with Keycloak SSO Integration Guide

## Prerequisites

- Docker installed and running
- Basic familiarity with command line and YAML files

## Installation Steps

### 1. Generate Synapse Configuration

Run the following command to generate initial configuration:

```bash
docker run -it --rm \
  --mount type=volume,src=synapse-data,dst=/data \
  -e SYNAPSE_SERVER_NAME=my.matrix.host \
  -e SYNAPSE_REPORT_STATS=yes \
  matrixdotorg/synapse:latest generate
```

### 2. Launch Synapse Container

```bash
docker run -d \
  --name synapse \
  --mount type=volume,src=synapse-data,dst=/data \
  -p 8008:8008 \
  matrixdotorg/synapse:latest
```

### 3. Start Services with Docker Compose

```bash
docker compose up -d
```

If you encounter issues, try specifying the compose file explicitly:

```bash
docker compose -f docker-compose.yml up -d
```

## Configuration

1. Locate the `homeserver.yaml` configuration file:
   - Access through Docker Desktop
   - Navigate to Volumes → synapse-data

2. Add the following configuration at the end of the `homeserver.yaml` file:

```yaml

# Keycloak OIDC Configuration
oidc_providers:
  - idp_id: keycloak
    idp_name: "SSO"
    allow_insecure: true
    skip_verification: true
    issuer: "http://host.docker.internal:8080/realms/master"
    client_id: "synapse"
    client_secret: "sRLeczfbwWE59rb0DXIBmnD2BYuoe6OG"
    scopes: ["openid", "profile", "email"]
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.name }}"
    backchannel_logout_enabled: true
```

## Keycloak Configuration

1. Create a new realm named "synapse"
2. Create a client with these settings:
   - Client ID: `synapse`
   - Client Secret: `sRLeczfbwWE59rb0DXIBmnD2BYuoe6OG`
   - Issuer URL: `http://host.docker.internal:8080/realms/master`

## Troubleshooting

- If containers crash, restart them through Docker Desktop
- Refer to the official documentation for additional guidance:  
  [Synapse OIDC Documentation](https://element-hq.github.io/synapse/latest/openid.html#keycloak)
