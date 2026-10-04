The storage server has no authentication unless the operator configures it. When configured, every request needs an `Authorization: Bearer <token>` header and answers 401 without a valid token, and a read-only caller gets 403 on every write route. Without it, any client that can reach the server can read every model and write to it. Run it on a trusted network or behind an operator-supplied gateway.

| Route | Page |
|---|---|
| `GET /models` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/versions` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/commits` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/commit` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/git/history` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/git/at/{version}` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/pull` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/push` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/data-slices` | [Model routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/data-slices/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}/data-slices/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/query-templates` | [Model routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/query-templates` | [Model routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/query-templates/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}/query-templates/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/aggregations` | [Model routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/aggregations/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}/aggregations/{itemId}` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files/add` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files/remove` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/manifest-files` | [Model routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/manifest-files/import` | [Model routes](/docs/sightline/reference/api/models) |
| `GET /discovery/models` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/findings` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/bindings` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/external-data` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/rules` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/attribute-sets` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/schemes` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/manifest-fragments` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/findings` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/coverage` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/covering-causes` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/vectors` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/vector-staleness` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/vector-coverage` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/assessments/{id}` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/assessments/{id}/ops` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/assessments/{id}/run` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/assessments/{id}/promote` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/findings/patches` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/packages/{id}/portfolio` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/packages/{id}/portfolio/export` | [Discovery routes](/docs/sightline/reference/api/discovery) |
| `POST /exports` | [Export routes](/docs/sightline/reference/api/exports) |
| `GET /exports/{id}` | [Export routes](/docs/sightline/reference/api/exports) |
| `DELETE /exports/{id}` | [Export routes](/docs/sightline/reference/api/exports) |
| `GET /exports/{id}/file` | [Export routes](/docs/sightline/reference/api/exports) |

## Errors

A failed request returns:

```json
{
  "error": "a message describing what went wrong"
}
```
