The storage server has no authentication unless the operator configures it. When configured, every request needs an `Authorization: Bearer <token>` header and answers 401 without a valid token, and a read-only caller gets 403 on every write route. Without it, any client that can reach the server can read every model and write to it. Run it on a trusted network or behind an operator-supplied gateway.

| Route | Page |
|---|---|
| `GET /models` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}` | [Model Routes](/docs/sightline/reference/api/models) |
| `PATCH /models/{id}` | [Model Routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/versions` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/events` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/commits` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/outputs` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/notes` | [Model Routes](/docs/sightline/reference/api/models) |
| `PATCH /models/{id}/notes` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/commit` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/git/history` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/git/at/{version}` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/pull` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/git/push` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/data-slices` | [Model Routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/data-slices/{itemId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}/data-slices/{itemId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{id}/query-templates` | [Model Routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/query-templates` | [Model Routes](/docs/sightline/reference/api/models) |
| `PUT /models/{id}/query-templates/{itemId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{id}/query-templates/{itemId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files/add` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/template-files/remove` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/manifest-files` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/manifest-files/import` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/findings/{findingId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `PATCH /models/{modelId}/findings/{findingId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/findings/{findingId}/journal` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/findings/{findingId}/usages` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/findings/{findingId}/members` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{modelId}/findings` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/rule-packages` | [Model Routes](/docs/sightline/reference/api/models) |
| `PUT /models/{modelId}/rule-packages/{packageId}/rules/{ruleId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `PUT /models/{modelId}/uses` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{modelId}/finding-runs` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /models/{modelId}/diagnostics` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{id}/move` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{modelId}/findings/{findingId}/links` | [Model Routes](/docs/sightline/reference/api/models) |
| `DELETE /models/{modelId}/findings/{findingId}/links/{linkId}` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{modelId}/findings/{findingId}/approvals` | [Model Routes](/docs/sightline/reference/api/models) |
| `POST /models/{modelId}/findings/{findingId}/approvals/withdraw` | [Model Routes](/docs/sightline/reference/api/models) |
| `GET /session` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/models` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/bindings` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/external-data` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/rules` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/rule-packages` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/rule-packages/{packageId}/usage` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/attribute-sets` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/schemes` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /discovery/manifest-fragments` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/findings` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/coverage` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/covering-causes` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/tetra-scope` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/finding-links` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/causes/{causeId}/finding-links` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/risks` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/control-templates/{templateId}/impact` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/causes/{causeId}/controls/{controlId}/impact` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/outcomes/{outcomeId}/controls/{controlId}/impact` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /cross-app/models/{modelId}/risks/query` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /cross-app/models/{modelId}/risks/preview` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /cross-app/models/{modelId}/finding-inbox` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/assessments/{id}` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/assessments/{id}/workspace` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/assessments/{id}/ops` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/assessments/{id}/run` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/packages/{id}/assessments` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /metron/packages/{id}/versions` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/packages/{id}/portfolio` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /metron/packages/{id}/portfolio/export` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /folders` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /folders` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /edit-grants` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `GET /findings` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `PATCH /folders/{folder}` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `DELETE /folders/{folder}` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /folders/{folder}/move` | [Discovery Routes](/docs/sightline/reference/api/discovery) |
| `POST /exports` | [Export Routes](/docs/sightline/reference/api/exports) |
| `GET /exports/{id}` | [Export Routes](/docs/sightline/reference/api/exports) |
| `DELETE /exports/{id}` | [Export Routes](/docs/sightline/reference/api/exports) |
| `GET /exports/{id}/file` | [Export Routes](/docs/sightline/reference/api/exports) |

## Errors

A failed request returns:

```json
{
  "error": "a message describing what went wrong"
}
```
