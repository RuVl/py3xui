# Models

py3xui uses Pydantic models for API payload validation and parsing.

Core models include:
- `Inbound`
- `Client`
- `ClientDetails`
- `ServerStatus`
- `ServerConfig`

`Client` keeps fields it does not declare (`extra="allow"`) and returns them from `model_dump()`.
The panel replaces a client on `clients/update/{email}`, so dropping unknown fields on read would
erase them on the next update. `Client` also accepts the client record shape returned by
`clients/get/{email}`: `allowedIPs` as a comma-separated string, `createdAt` / `updatedAt`, and the
UUID in `uuid` while `id` holds the record ID.

`ClientDetails` is the full `clients/get/{email}` payload: `client`, `inbound_ids`,
`external_links`, `used_traffic` and `tunnel_allowed_ips`.

When the upstream panel response shape changes, py3xui adapts parsing logic while preserving stable public methods where possible.

## Clients API models (3x-ui v3.11.0)

- `ClientPage`: result of `get_list_paged`: `items` (`ClientSlim`), `total`, `filtered`, `page`, `page_size`, `summary` (`ClientsSummary`), `groups`.
- `ClientSlim`: compact row without credentials, it can not be passed to `update`.
- `ClientsSummary`: client counters over the whole panel; the email lists stop at 200 entries.
- `ClientTraffic`: traffic counters of a client.
- `ClientRenewalPreview`: result of `get_renewal_preview`.
- `ClientExternalLink`: external link or subscription, argument of `set_external_links`. `ClientDetails.external_links` items can be loaded with `ClientExternalLink.model_validate`.
- `ClientEntry`: `client`, `inbound_ids` and `traffic` (`ClientEntryTraffic`). Returned by `export_clients`, accepted by `import_clients` and `bulk_add`.
- `BulkResult`: `count` (created, adjusted, deleted or changed clients) and `skipped` (`BulkSkip` with `email` and `reason`).
- `BulkAttachResult` / `BulkDetachResult`: `attached` or `detached`, `skipped` and `errors` as lists of strings.
- `ClientGroup`: `name`, `client_count`, `traffic_used`, `up`, `down`.
- `ClientHwid`: registered device: `id`, `first_seen`, `last_seen`, `user_agent`, `device_os`, `os_version`, `device_model`, `fingerprint`.
- `ClientIpEntry`: `ip` and `timestamp` (Unix seconds).
## Inbounds API (3x-ui v3.11.0)

- `InboundOption`: lightweight inbound projection returned by `get_options()`.
- `InboundFallback`: fallback rule of a master inbound, used by `get_fallbacks()` and `set_fallbacks()`.
- `AmneziaWGReadiness`, `AmneziaWGBlocker`: host readiness for kernel-mode AmneziaWG.
- `InboundBulkDeleteResult`, `InboundBulkDeleteReport`: result of `bulk_delete()`.
- `InboundClientsDeleteResult`, `InboundClientsDeleteReport`: result of `delete_all_clients()`.
## Server API (3x-ui v3.11.0)

Models in `py3xui.server`:

- `Fail2banStatus`, `HistoryPoint`, `CpuHistoryPoint`, `XrayMetricsState`, `ObservatorySnapshot`
- `PanelUpdateInfo`, `PanelUpdateStatus`
- `Mldsa65KeyPair`, `Mlkem768KeyPair`, `VlessEncryptionAuth`, `EchCert`, `WebCertFiles`, `RealityScanResult`
- `XrayLogEntry`, `AmneziaWGLogs`, `AmneziaWGPeer`
- `NodeSummary`, `InboundClientIps`, `InboundClientIpEntry`
## Settings API (3x-ui v3.11.0)

- `AllSetting`: every panel setting, returned by `api.setting.get_all()` and accepted by `api.setting.update()`. Unknown keys are kept and sent back. The `has_*` flags are read-only and left out of `model_dump()`.
- `TgBotCapabilities`: what the Telegram bot can do, returned by `api.setting.get_tg_bot_capabilities()`.
- `ApiToken`: an API token, returned by `api.setting.get_api_tokens()` and `api.setting.add_api_token()`. `token` is filled only by `add_api_token()`. `created_at` is in Unix seconds and `expires_at` in Unix milliseconds.
## Xray API (3x-ui v3.11.0)

Models of `api.xray`, exported from `py3xui`:

- `XraySettings`, `GeodataSource`, `OutboundTraffic`
- `PiaAccount`, `PiaCountry`, `PiaRegion`, `PiaServer`, `PiaServers`, `PiaKey`
- `OutboundTestResult`, `OutboundTestEndpoint`, `OutboundTestEgress`, `BalancerStatus`, `RouteTestResult`
- `GeoFile`, `GeoCategory`, `GeoCategoryPage`, `GeoEntry`, `GeoEntryPage`, `GeodataTokenIssue`
- `OutboundSub` (keeps unknown fields, is sent back to the panel)
## Nodes API (3x-ui v3.11.0)

- `Node`: a remote panel, used for reads and writes. Unknown fields are kept and sent back on update. `api_token` is write-only: `None` keeps the stored token, `clear_api_token=True` removes it. A `null` `inboundTags` from the panel is parsed as an empty list.
- `NodeProbeResult`: result of `test()` and `probe()`. An unreachable node has the `offline` status and the reason in `error`, it does not raise.
- `NodeUpdateResult`: per-node result of `update_panel()`.
- `NodeRemoteInbound`: an inbound returned by `get_remote_inbounds()`.
- `NodeWebCertFiles`: result of `get_web_cert_files()`.
- `NodeHistoryPoint`: a point of `get_history()`.
