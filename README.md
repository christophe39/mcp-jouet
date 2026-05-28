# MCP Jouet OPEPARTNER

MCP minimal protégé par OIDCProxy (FastMCP) → Keycloak, pour valider la compatibilité avec les clients Claude.

## Stack

- **FastMCP 3.3.1** avec OIDCProxy
- **Keycloak** : realm `mcp`, client `mcp-jouet` (confidential)
- **Redis** : stockage chiffré des sessions/tokens (via FernetEncryptionWrapper)
- **Domaine** : `https://mcp-jouet.agnisolution.fr`

## Variables d'environnement (Coolify)

```
OIDC_CONFIG_URL=https://keycloak.agnisolution.fr/realms/mcp/.well-known/openid-configuration
OIDC_CLIENT_ID=mcp-jouet
OIDC_CLIENT_SECRET=<secret par Christophe>
MCP_BASE_URL=https://mcp-jouet.agnisolution.fr
REDIS_URL=<Redis URL interne, par Christophe>
JWT_SIGNING_KEY=<généré par Christophe>
FERNET_SECRET=<généré par Christophe>
```

## Healthcheck Coolify

- Path : `/.well-known/oauth-protected-resource/mcp`
- Port : 8080
- Start period : 30s
- Return code attendu : 200

⚠️ NE PAS utiliser `/mcp` qui retourne 401 sans token.

## Critères go/no-go (Phase 3.5)

1. Container healthy
2. Protected resource metadata accessible
3. **Authorization server metadata sur le domaine du MCP** (pas keycloak.agnisolution.fr)
4. Endpoint `/mcp` exige auth (401 sans token)

Le critère 3 est LE plus critique : si les endpoints OAuth sont bien servis par `mcp-jouet.agnisolution.fr` (et pas Keycloak directement), on a contourné le bug Claude.ai #82.

## Outil de test

```python
@mcp.tool
async def hello_world(name: str = "monde") -> str:
    """Outil de test : renvoie un salut avec l'utilisateur authentifié."""
    token = get_access_token()
    user_id = token.claims.get("sub", "anonyme")
    scopes = token.claims.get("scope", "")
    return f"Bonjour {name} ! Le MCP jouet fonctionne. Utilisateur: {user_id}, Scopes: {scopes}"
```
