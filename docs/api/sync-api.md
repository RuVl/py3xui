# Synchronous API

## Create Client

```python
from py3xui import Api

api = Api("http://your-host:2053", "username", "password")
api.login()
```

## Basic Health Check

```python
status = api.server.get_status()
print("CPU:", status.cpu)
print("Memory:", status.mem.current, "/", status.mem.total)
```

## Typical Flow

```python
inbounds = api.inbound.get_list()
client = api.client.get_by_email("user@example.com")
online = api.client.online()
status = api.server.get_status()
```

## Client Details and Safe Updates

`get_details()` returns the full client payload: the client, every inbound it is attached to,
external links, used traffic and tunnel addresses. The panel replaces the client on update, so
edit the returned `details.client` and pass it back: all its fields, including ones py3xui does
not declare, are sent with the update.

```python
details = api.client.get_details("user@example.com")

if details:
	print(details.inbound_ids, details.tunnel_allowed_ips)
	details.client.total_gb = 50 * 1024 * 1024 * 1024
	api.client.update(details.client.uuid, details.client)
```

## Client Lifecycle Example

```python
from uuid import uuid4
from py3xui import Client

inbound_id = 1
email = "demo@example.com"

api.client.add(inbound_id, [Client(id=str(uuid4()), email=email, enable=True)])
client = api.client.get_by_email(email)

if client:
	client.total_gb = 10 * 1024 * 1024 * 1024
	api.client.update(str(client.id), client)

api.client.reset_ips(email)
api.client.reset_stats(inbound_id, email)
api.client.delete(inbound_id, str(client.id) if client else email)
```

## Error Handling Pattern

When the panel answers with `success: false`, py3xui raises `XuiApiError`. It is a subclass of
`ValueError` and carries the panel's `msg` and `obj` fields.

```python
from py3xui import XuiApiError

try:
	api.login()
	print(api.server.get_status())
except XuiApiError as exc:
	print("API response error:", exc.msg)
except ValueError as exc:
	print("Client-side error:", exc)
except Exception as exc:
	print("Unexpected failure:", exc)
```

## Notes

- Calls are blocking.
- Suitable for scripts, cron jobs, and simple service integrations.

## More Examples

- [Common Tasks](../examples/common-tasks.md)
- [Synchronous Recipes](../examples/sync-recipes.md)

## Clients API (3x-ui v3.11.0)

All methods live on `api.client`.

- `get_list_paged(page, page_size, *, search, statuses, protocols, inbound_ids, groups, sort, order, expiry_from, expiry_to, usage_from, usage_to, auto_renew, has_tg_id, has_comment)`: one page of clients, filtered and sorted by the panel.
- `get_details_by_sub_id(sub_id)`: full details of all clients with a subscription ID. The panel reports an unknown ID as an error, so this raises `ValueError`.
- `get_details_by_tg_id(tg_id)`: full details of all clients with a Telegram ID.
- `get_renewal_preview(expiry_time, *, reset, reset_day, reset_weekday, reset_max, reset_count)`: auto-renewal dates, nothing is saved.
- `get_traffic(email)`: traffic counters of a client, `None` if there are none.
- `get_sub_links(sub_id)`: share links of a subscription.
- `get_links(email)`: share links of a client.
- `get_happ_link(client_id)`: encrypted Happ link, by client record ID.
- `attach(email, inbound_ids)`: attaches a client to more inbounds.
- `detach(email, inbound_ids)`: detaches a client from inbounds, the client is kept.
- `set_external_links(email, links)`: replaces all external links and subscriptions of a client.
- `delete_orphans()`: deletes clients without inbounds, returns the number of deleted clients.
- `export_clients()`: all clients with their inbounds and usage counters.
- `import_clients(entries)`: imports the result of `export_clients`, existing emails are skipped.
- `bulk_adjust(emails, add_days, add_bytes, *, flow, limit_hwid, ad_tag)`: shifts the expiry and traffic limit of many clients, optionally sets flow, device limit and ad tag.
- `bulk_enable(emails)` / `bulk_disable(emails)`: enables or disables many clients.
- `bulk_delete(emails, keep_traffic)`: deletes many clients.
- `bulk_add(entries)`: creates many clients, each with its own inbounds.
- `bulk_attach(emails, inbound_ids)` / `bulk_detach(emails, inbound_ids)`: attaches many clients to inbounds or detaches them.
- `bulk_reset_stats(emails)`: resets the traffic counters of many clients, returns their number.
- `get_groups()`: all client groups with member counts and traffic.
- `get_group_emails(name)`: emails of the clients of a group.
- `add_group(name)`: creates an empty group.
- `rename_group(old_name, new_name)`: renames a group, returns the number of affected clients.
- `delete_group(name)`: deletes a group, the clients are kept. Returns the number of affected clients.
- `reset_group_stats(name)`: resets the traffic counter of a group, client counters are not changed.
- `bulk_add_to_group(group, emails)` / `bulk_remove_from_group(emails)`: sets or clears the group of many clients, returns the number of affected clients.
- `update_traffic(email, up, down)`: sets the traffic counters of a client (absolute values).
- `get_hwids(email)`: devices registered for a client.
- `delete_hwids(email)` / `delete_hwid(email, hwid_id)`: deletes all devices of a client or one of them.
- `online_by_guid()`: online emails by panel GUID.
- `get_ips_by_guid()`: client IPs by panel GUID and email.
- `get_active_inbounds()`: inbound tags with recent traffic by panel GUID.
- `get_last_online()`: last online time in milliseconds by email.
## Inbounds API (3x-ui v3.11.0)

Additional methods of `api.inbound`:

- `get_list_slim()`: list inbounds with settings.clients stripped to email, enable and comment.
- `get_options()`: lightweight inbound projection (`list[InboundOption]`).
- `get_amneziawg_kernel_readiness()`: check whether the host can run kernel-mode AmneziaWG (`AmneziaWGReadiness`).
- `get_all_links()`: share links of all clients of all inbounds (`list[str]`).
- `bulk_delete(inbound_ids)`: delete several inbounds (`InboundBulkDeleteResult`).
- `set_enable(inbound_id, enable)`: enable or disable an inbound without sending its settings.
- `set_sub_sort_index(inbound_id, index)`: set the subscription sort order (0 is rejected by the panel).
- `reset_stats_by_id(inbound_id)`: reset the traffic of one inbound.
- `delete_all_clients(inbound_id)`: delete all clients and keep the inbound (`InboundClientsDeleteResult`).
- `import_inbound(inbound)`: import an `Inbound` or a raw JSON string (lossless), returns the created `Inbound`.
- `push_client_traffics(master_guid, traffics)`: push aggregated client traffic from a master panel.
- `get_fallbacks(inbound_id)`: fallback rules of a master inbound (`list[InboundFallback]`).
- `set_fallbacks(inbound_id, fallbacks)`: replace the fallback rules of a master inbound.
## Server API (3x-ui v3.11.0)

- `api.server.get_fail2ban_status()`: availability of Fail2ban, which per-client IP limits depend on.
- `api.server.get_cpu_history(bucket)`: legacy aggregated CPU history.
- `api.server.get_history(metric, bucket)`: aggregated history of a system metric.
- `api.server.get_xray_metrics_state()`: state of the Xray runtime metrics collection.
- `api.server.get_xray_metrics_history(metric, bucket)`: aggregated history of an Xray metric.
- `api.server.get_xray_observatory()`: latest Xray observatory results per outbound.
- `api.server.get_xray_observatory_history(tag, bucket)`: delay history of an observatory outbound.
- `api.server.get_panel_update_info()`: current and latest version of the panel.
- `api.server.get_update_status()`: outcome of the most recent panel self-update.
- `api.server.update_panel(dev=None)`: starts a panel self-update and returns the run ID.
- `api.server.set_update_channel(dev)`: switches the update channel between stable and dev.
- `api.server.stop_xray()` / `api.server.restart_xray()`: Xray service control.
- `api.server.update_geofile(file_name=None)`: updates one geofile, or all of them without an argument.
- `api.server.generate_uuid()`: generates a new UUID.
- `api.server.generate_mldsa65_keys()` / `api.server.generate_mlkem768_keys()`: generate post-quantum key pairs.
- `api.server.generate_vless_encryption()`: generates VLESS encryption options.
- `api.server.generate_ech_cert(sni)`: generates an ECH key set.
- `api.server.get_web_cert_files()`: paths of the web TLS files of the panel.
- `api.server.get_cert_hash(cert_file=None, cert_content=None)`: SHA-256 hash of a certificate for pinning.
- `api.server.get_remote_cert_hash(server, allow_private=False)`: SHA-256 hash of the certificate of a remote server.
- `api.server.scan_reality_target(target, sni=None, xver=0, allow_private=False)`: probes a REALITY target.
- `api.server.scan_reality_targets(targets=None)`: probes several REALITY targets.
- `api.server.get_logs(count, level="info", syslog=False)`: last lines of the panel log.
- `api.server.get_xray_logs(count, keyword="", show_direct=True, show_blocked=True, show_proxy=True)`: last lines of the Xray access log.
- `api.server.get_amneziawg_logs(count, keyword="")`: live AmneziaWG activity and log lines.
- `api.server.get_migration(save_path)`: saves a cross-engine migration file.
- `api.server.import_db(file_path, keep_host_settings=True)`: restores the panel database from a file.
- `api.server.get_descendants()`: summaries of the nodes managed by the panel.
- `api.server.get_client_ips()`: recently active IP addresses of all the clients.
- `api.server.push_client_ips(client_ips)`: merges recently active IP addresses into the panel database.
## Settings API (3x-ui v3.11.0)

`api.setting` covers `panel/api/setting/*`.

```python
settings = api.setting.get_all()
settings.sub_title = "My subscription"
api.setting.update(settings)
```

- Reads: `get_all`, `get_default_settings`, `get_factory_defaults`, `get_default_xray_config`, `get_tg_bot_capabilities`.
- Changes: `update`, `update_user`, `restart_panel`, `validate_regex`.
- Tests: `test_smtp` (returns the result, also on failure), `test_tg_bot`, `test_discord`.

`update` replaces every setting, so start from `get_all()`. Secrets come back blank and a blank secret keeps the stored value; pass `clear_*` to remove one.

API tokens: `get_api_tokens`, `add_api_token` (the only call that returns the token value), `delete_api_token` and `set_api_token_enabled`. The last two need the stored scope of the token in `expected_scope`; a mismatch is rejected by the panel.
## Xray API (3x-ui v3.11.0)

`api.xray` wraps `/panel/api/xray/*`.

```python
settings = api.xray.get_settings()
traffic = api.xray.get_outbounds_traffic()
```

| Method | Endpoint | Returns |
|---|---|---|
| `get_settings()` | `POST /panel/api/xray/` | `XraySettings` |
| `get_default_config()` | `GET /panel/api/xray/getDefaultJsonConfig` | `dict` |
| `get_outbounds_traffic()` | `GET /panel/api/xray/getOutboundsTraffic` | `list[OutboundTraffic]` |
| `get_result()` | `GET /panel/api/xray/getXrayResult` | `str` |
| `update(config, outbound_test_url=None)` | `POST /panel/api/xray/update` | `None` |
| `reset_outbound_stats(tag)` | `POST /panel/api/xray/resetOutboundsTraffic` | `None` |
| `warp_get_data()` | `POST /panel/api/xray/warp/data` | `dict` |
| `warp_delete()` | `POST /panel/api/xray/warp/del` | `None` |
| `warp_get_config()` | `POST /panel/api/xray/warp/config` | `dict` |
| `warp_register(private_key, public_key)` | `POST /panel/api/xray/warp/reg` | `dict` |
| `warp_register_masque()` | `POST /panel/api/xray/warp/regMasque` | `dict` |
| `warp_change_ip()` | `POST /panel/api/xray/warp/changeIp` | `dict` |
| `warp_set_license(license_key)` | `POST /panel/api/xray/warp/license` | `dict` |
| `warp_set_interval(interval)` | `POST /panel/api/xray/warp/interval` | `None` |
| `nord_get_countries()` | `POST /panel/api/xray/nord/countries` | `list[dict]` |
| `nord_get_servers(country_id)` | `POST /panel/api/xray/nord/servers` | `dict` |
| `nord_register(token)` | `POST /panel/api/xray/nord/reg` | `dict` |
| `nord_set_key(key)` | `POST /panel/api/xray/nord/setKey` | `dict` |
| `nord_get_data()` | `POST /panel/api/xray/nord/data` | `dict` |
| `nord_delete()` | `POST /panel/api/xray/nord/del` | `None` |
| `pia_get_countries()` | `POST /panel/api/xray/pia/countries` | `list[PiaCountry]` |
| `pia_get_servers(country_code)` | `POST /panel/api/xray/pia/servers` | `PiaServers` |
| `pia_register(username, password)` | `POST /panel/api/xray/pia/reg` | `PiaAccount` |
| `pia_get_data()` | `POST /panel/api/xray/pia/data` | `PiaAccount \| None` |
| `pia_delete()` | `POST /panel/api/xray/pia/del` | `None` |
| `pia_add_key(hostname)` | `POST /panel/api/xray/pia/addKey` | `PiaKey` |
| `test_outbound(outbound, all_outbounds=None, mode=None)` | `POST /panel/api/xray/testOutbound` | `OutboundTestResult` |
| `test_outbounds(outbounds, all_outbounds=None, mode=None)` | `POST /panel/api/xray/testOutbounds` | `list[OutboundTestResult]` |
| `get_balancer_status(tags)` | `POST /panel/api/xray/balancerStatus` | `dict[str, BalancerStatus]` |
| `set_balancer_override(balancer_tag, target="")` | `POST /panel/api/xray/balancerOverride` | `None` |
| `test_route(domain=None, ip=None, port=None, network=None, inbound_tag=None, protocol=None, email=None)` | `POST /panel/api/xray/routeTest` | `RouteTestResult` |
| `get_geodata_files()` | `GET /panel/api/xray/geodata/files` | `list[GeoFile]` |
| `get_geodata_categories(file, query=None, offset=None, limit=None)` | `GET /panel/api/xray/geodata/categories` | `GeoCategoryPage` |
| `get_geodata_entries(file, code, query=None, offset=None, limit=None)` | `GET /panel/api/xray/geodata/entries` | `GeoEntryPage` |
| `validate_geodata(tokens, kind=None)` | `POST /panel/api/xray/geodata/validate` | `list[GeodataTokenIssue]` |
| `get_outbound_subs()` | `GET /panel/api/xray/outbound-subs` | `list[OutboundSub]` |
| `add_outbound_sub(sub)` | `POST /panel/api/xray/outbound-subs` | `OutboundSub` |
| `update_outbound_sub(sub_id, sub)` | `POST /panel/api/xray/outbound-subs/{id}` | `None` |
| `delete_outbound_sub(sub_id)` | `POST /panel/api/xray/outbound-subs/{id}/del` | `None` |
| `refresh_outbound_sub(sub_id)` | `POST /panel/api/xray/outbound-subs/{id}/refresh` | `list[dict]` |
| `move_outbound_sub(sub_id, direction)` | `POST /panel/api/xray/outbound-subs/{id}/move` | `None` |
| `parse_outbound_sub(url, user_agent=None, allow_private=False, allow_insecure=False)` | `POST /panel/api/xray/outbound-subs/parse` | `list[dict]` |
