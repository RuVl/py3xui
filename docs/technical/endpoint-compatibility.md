# Endpoint Compatibility

3x-ui endpoint naming changed in modern v3 builds, especially for client operations.

The 0.7.0 line aligns client routes to `panel/api/clients/*` and keeps compatibility wrappers in Python method signatures where feasible.

Examples:
- Add client: `panel/api/clients/add`
- Update client: `panel/api/clients/update/{email}`
- Client details: `panel/api/clients/get/{email}` (`get_details`, full payload; `get_by_email` keeps the 0.7.0 shape)
- Online clients: `panel/api/clients/onlines`

Compatibility behavior is implemented in both sync and async clients.

## Clients API (3x-ui v3.11.0)

- `GET panel/api/clients/list/paged`: `get_list_paged`
- `GET panel/api/clients/get/subId/{subId}`: `get_details_by_sub_id`
- `GET panel/api/clients/get/tgId/{tgId}`: `get_details_by_tg_id`
- `POST panel/api/clients/renewalPreview`: `get_renewal_preview`
- `GET panel/api/clients/traffic/{email}`: `get_traffic`
- `GET panel/api/clients/subLinks/{subId}`: `get_sub_links`
- `GET panel/api/clients/links/{email}`: `get_links`
- `POST panel/api/clients/happLink/{id}`: `get_happ_link`
- `POST panel/api/clients/{email}/attach`: `attach`
- `POST panel/api/clients/{email}/detach`: `detach`
- `POST panel/api/clients/{email}/externalLinks`: `set_external_links`
- `POST panel/api/clients/delOrphans`: `delete_orphans`
- `GET panel/api/clients/export`: `export_clients`
- `POST panel/api/clients/import`: `import_clients`
- `POST panel/api/clients/bulkAdjust`: `bulk_adjust`
- `POST panel/api/clients/bulkEnable`: `bulk_enable`
- `POST panel/api/clients/bulkDisable`: `bulk_disable`
- `POST panel/api/clients/bulkDel`: `bulk_delete`
- `POST panel/api/clients/bulkCreate`: `bulk_add`
- `POST panel/api/clients/bulkAttach`: `bulk_attach`
- `POST panel/api/clients/bulkDetach`: `bulk_detach`
- `POST panel/api/clients/bulkResetTraffic`: `bulk_reset_stats`
- `GET panel/api/clients/groups`: `get_groups`
- `GET panel/api/clients/groups/{name}/emails`: `get_group_emails`
- `POST panel/api/clients/groups/create`: `add_group`
- `POST panel/api/clients/groups/rename`: `rename_group`
- `POST panel/api/clients/groups/delete`: `delete_group`
- `POST panel/api/clients/groups/resetTraffic`: `reset_group_stats`
- `POST panel/api/clients/groups/bulkAdd`: `bulk_add_to_group`
- `POST panel/api/clients/groups/bulkRemove`: `bulk_remove_from_group`
- `POST panel/api/clients/updateTraffic/{email}`: `update_traffic`
- `POST panel/api/clients/hwids/{email}`: `get_hwids`
- `DELETE panel/api/clients/hwids/{email}`: `delete_hwids`
- `DELETE panel/api/clients/hwids/{email}/{id}`: `delete_hwid`
- `POST panel/api/clients/onlinesByGuid`: `online_by_guid`
- `POST panel/api/clients/clientIpsByGuid`: `get_ips_by_guid`
- `POST panel/api/clients/activeInbounds`: `get_active_inbounds`
- `POST panel/api/clients/lastOnline`: `get_last_online`
## Inbounds API (3x-ui v3.11.0)

- `GET panel/api/inbounds/list/slim`
- `GET panel/api/inbounds/options`
- `GET panel/api/inbounds/amneziawgKernelReadiness`
- `GET panel/api/inbounds/allLinks`
- `POST panel/api/inbounds/bulkDel`
- `POST panel/api/inbounds/setEnable/{id}`
- `POST panel/api/inbounds/{id}/subSortIndex`
- `POST panel/api/inbounds/{id}/resetTraffic`
- `POST panel/api/inbounds/{id}/delAllClients`
- `POST panel/api/inbounds/import` (form field `data` with the inbound JSON)
- `POST panel/api/inbounds/pushClientTraffics`
- `GET panel/api/inbounds/{id}/fallbacks`
- `POST panel/api/inbounds/{id}/fallbacks`
## Server API (3x-ui v3.11.0)

Endpoints used by `api.server`:

- `GET panel/api/server/fail2banStatus`
- `GET panel/api/server/cpuHistory/{bucket}`
- `GET panel/api/server/history/{metric}/{bucket}`
- `GET panel/api/server/xrayMetricsState`
- `GET panel/api/server/xrayMetricsHistory/{metric}/{bucket}`
- `GET panel/api/server/xrayObservatory`
- `GET panel/api/server/xrayObservatoryHistory/{tag}/{bucket}`
- `GET panel/api/server/getPanelUpdateInfo`
- `GET panel/api/server/getUpdateStatus`
- `POST panel/api/server/updatePanel`
- `POST panel/api/server/setUpdateChannel`
- `POST panel/api/server/stopXrayService`
- `POST panel/api/server/restartXrayService`
- `POST panel/api/server/updateGeofile/{fileName}`
- `GET panel/api/server/getNewUUID`
- `GET panel/api/server/getNewmldsa65`
- `GET panel/api/server/getNewmlkem768`
- `GET panel/api/server/getNewVlessEnc`
- `GET panel/api/server/getWebCertFiles`
- `POST panel/api/server/getNewEchCert`
- `POST panel/api/server/getCertHash`
- `POST panel/api/server/getRemoteCertHash`
- `POST panel/api/server/scanRealityTarget`
- `POST panel/api/server/scanRealityTargets`
- `POST panel/api/server/logs/{count}`
- `POST panel/api/server/xraylogs/{count}`
- `POST panel/api/server/amneziawglogs/{count}`
- `GET panel/api/server/getMigration`
- `POST panel/api/server/importDB`
- `GET panel/api/server/descendants`
- `GET panel/api/server/clientIps`
- `POST panel/api/server/clientIps`
## Settings API (3x-ui v3.11.0)

Sync and async `api.setting` use JSON bodies on these routes:

- `POST panel/api/setting/all`, `defaultSettings`, `factoryDefaults`, `update`, `validateRegex`, `updateUser`, `restartPanel`
- `POST panel/api/setting/testSmtp`, `testTgBot`, `tgBotCapabilities`, `testDiscord`
- `GET panel/api/setting/getDefaultJsonConfig`, `apiTokens`
- `POST panel/api/setting/apiTokens/create`, `apiTokens/delete/{id}`, `apiTokens/setEnabled/{id}`

`testSmtp` adds a `stage` field to the response, so `test_smtp()` returns it instead of raising on failure.
## Xray API (3x-ui v3.11.0)

- `POST xray/` returns its payload as a JSON string in `obj`; `get_settings()` parses it into `XraySettings`.
- `xray/update` replaces the outbound test URL on every call: when `outboundTestUrl` is absent the panel resets it to the default one.
- `update()` and the outbound test methods take the Xray JSON as a string, exactly as the form fields expect it.
- `warp/*` and `nord/*` return JSON strings in `obj`; the methods parse them, an empty value gives an empty `dict`. `pia/*` return plain objects.
- The `warp`, `nord` and `pia` routes take an `{action}` path parameter; every action has its own method with typed arguments.
- `balancerStatus` reports `selected: null` for a balancer that is not running; `BalancerStatus.selected` is an empty list then.
- `outbound-subs/{id}` also accepts DELETE; `delete_outbound_sub()` uses the POST `.../{id}/del` alias.
- `outbound-subs/{id}` replaces the editable fields: fields which are absent in the form get their defaults.
- Creating a subscription only stores it. Call `refresh_outbound_sub()` to fetch its outbounds.
## Nodes API (3x-ui v3.11.0)

All 16 `panel/api/nodes/*` routes are covered by `api.node` in both clients. All bodies are JSON.

- `update` sends the whole node, because the panel replaces it. An omitted `apiToken` keeps the stored token.
- `add`, `update`, `test`, `certFingerprint` and `inbounds` share one request contract: `name`, `address` and `port` are required by the panel on all of them.
- `get_history` accepts the metrics `cpu`, `mem`, `netUp`, `netDown` and the buckets 2, 30, 60, 180, 360, 720, 1440, 2880, 10080, as validated by the panel. The OpenAPI description of this route lists fewer metrics and different buckets.
## Hosts API (3x-ui v3.11.0)

| Endpoint | Method |
|---|---|
| `GET panel/api/hosts/list` | `api.host.get_list()` |
| `GET panel/api/hosts/raw` | `api.host.get_raw_list()` |
| `GET panel/api/hosts/get/{groupId}` | `api.host.get_by_id(group_id)` |
| `GET panel/api/hosts/byInbound/{inboundId}` | `api.host.get_by_inbound(inbound_id)` |
| `GET panel/api/hosts/tags` | `api.host.get_tags()` |
| `POST panel/api/hosts/add` | `api.host.add(host)` |
| `POST panel/api/hosts/update/{groupId}` | `api.host.update(group_id, host)` |
| `POST panel/api/hosts/del/{groupId}` | `api.host.delete(group_id)` |
| `POST panel/api/hosts/setEnable/{groupId}` | `api.host.set_enable(group_id, enable)` |
| `POST panel/api/hosts/reorder` | `api.host.reorder(group_ids)` |
| `POST panel/api/hosts/bulk/add` | `api.host.bulk_add(host)` |
| `POST panel/api/hosts/bulk/setEnable` | `api.host.bulk_set_enable(group_ids, enable)` |
| `POST panel/api/hosts/bulk/del` | `api.host.bulk_delete(group_ids)` |
## Sub Balancers API (3x-ui v3.11.0)

- `GET panel/api/sub-balancers`
- `POST panel/api/sub-balancers`
- `POST panel/api/sub-balancers/{id}`
- `POST panel/api/sub-balancers/{id}/del`

Create and update read form fields only: `inboundIds` as repeated keys and `memberWeights` as one JSON object. JSON bodies lose `inboundIds` and are rejected. `DELETE panel/api/sub-balancers/{id}` has the same handler as the POST alias, and py3xui uses the alias.
