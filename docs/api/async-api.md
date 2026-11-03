# Asynchronous API

## Create Client

```python
from py3xui import AsyncApi

api = AsyncApi("http://your-host:2053", "username", "password")
await api.login()
```

## Basic Health Check

```python
status = await api.server.get_status()
print("CPU:", status.cpu)
print("Memory:", status.mem.current, "/", status.mem.total)
```

## Typical Flow

```python
inbounds = await api.inbound.get_list()
client = await api.client.get_by_email("user@example.com")
online = await api.client.online()
status = await api.server.get_status()
```

## Client Details and Safe Updates

`get_details()` returns the full client payload: the client, every inbound it is attached to,
external links, used traffic and tunnel addresses. The panel replaces the client on update, so
edit the returned `details.client` and pass it back: all its fields, including ones py3xui does
not declare, are sent with the update.

```python
details = await api.client.get_details("user@example.com")

if details:
	print(details.inbound_ids, details.tunnel_allowed_ips)
	details.client.total_gb = 50 * 1024 * 1024 * 1024
	await api.client.update(details.client.uuid, details.client)
```

## Concurrent Reads Example

```python
import asyncio

status_task = api.server.get_status()
inbounds_task = api.inbound.get_list()
online_task = api.client.online()

status, inbounds, online = await asyncio.gather(
	status_task,
	inbounds_task,
	online_task,
)

print(status.cpu, len(inbounds), len(online))
```

## Error Handling Pattern

When the panel answers with `success: false`, py3xui raises `XuiApiError`. It is a subclass of
`ValueError` and carries the panel's `msg` and `obj` fields.

```python
from py3xui import XuiApiError

try:
	await api.login()
	print(await api.server.get_status())
except XuiApiError as exc:
	print("API response error:", exc.msg)
except ValueError as exc:
	print("Client-side error:", exc)
except Exception as exc:
	print("Unexpected failure:", exc)
```

## Notes

- Use inside an event loop.
- Best for concurrent integrations and high-throughput workloads.

## More Examples

- [Common Tasks](../examples/common-tasks.md)
- [Asynchronous Recipes](../examples/async-recipes.md)

## Clients API (3x-ui v3.11.0)

All methods live on `api.client` and match the sync API in names, arguments and return types.

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

Additional methods of `api.inbound` (same arguments and return types as the sync client, awaited):

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

- `await api.server.get_fail2ban_status()`: availability of Fail2ban, which per-client IP limits depend on.
- `await api.server.get_cpu_history(bucket)`: legacy aggregated CPU history.
- `await api.server.get_history(metric, bucket)`: aggregated history of a system metric.
- `await api.server.get_xray_metrics_state()`: state of the Xray runtime metrics collection.
- `await api.server.get_xray_metrics_history(metric, bucket)`: aggregated history of an Xray metric.
- `await api.server.get_xray_observatory()`: latest Xray observatory results per outbound.
- `await api.server.get_xray_observatory_history(tag, bucket)`: delay history of an observatory outbound.
- `await api.server.get_panel_update_info()`: current and latest version of the panel.
- `await api.server.get_update_status()`: outcome of the most recent panel self-update.
- `await api.server.update_panel(dev=None)`: starts a panel self-update and returns the run ID.
- `await api.server.set_update_channel(dev)`: switches the update channel between stable and dev.
- `await api.server.stop_xray()` / `await api.server.restart_xray()`: Xray service control.
- `await api.server.update_geofile(file_name=None)`: updates one geofile, or all of them without an argument.
- `await api.server.generate_uuid()`: generates a new UUID.
- `await api.server.generate_mldsa65_keys()` / `await api.server.generate_mlkem768_keys()`: generate post-quantum key pairs.
- `await api.server.generate_vless_encryption()`: generates VLESS encryption options.
- `await api.server.generate_ech_cert(sni)`: generates an ECH key set.
- `await api.server.get_web_cert_files()`: paths of the web TLS files of the panel.
- `await api.server.get_cert_hash(cert_file=None, cert_content=None)`: SHA-256 hash of a certificate for pinning.
- `await api.server.get_remote_cert_hash(server, allow_private=False)`: SHA-256 hash of the certificate of a remote server.
- `await api.server.scan_reality_target(target, sni=None, xver=0, allow_private=False)`: probes a REALITY target.
- `await api.server.scan_reality_targets(targets=None)`: probes several REALITY targets.
- `await api.server.get_logs(count, level="info", syslog=False)`: last lines of the panel log.
- `await api.server.get_xray_logs(count, keyword="", show_direct=True, show_blocked=True, show_proxy=True)`: last lines of the Xray access log.
- `await api.server.get_amneziawg_logs(count, keyword="")`: live AmneziaWG activity and log lines.
- `await api.server.get_migration(save_path)`: saves a cross-engine migration file.
- `await api.server.import_db(file_path, keep_host_settings=True)`: restores the panel database from a file.
- `await api.server.get_descendants()`: summaries of the nodes managed by the panel.
- `await api.server.get_client_ips()`: recently active IP addresses of all the clients.
- `await api.server.push_client_ips(client_ips)`: merges recently active IP addresses into the panel database.
## Settings API (3x-ui v3.11.0)

`api.setting` covers `panel/api/setting/*`, with the same methods and arguments as the sync client.

```python
settings = await api.setting.get_all()
settings.sub_title = "My subscription"
await api.setting.update(settings)
```

- Reads: `get_all`, `get_default_settings`, `get_factory_defaults`, `get_default_xray_config`, `get_tg_bot_capabilities`.
- Changes: `update`, `update_user`, `restart_panel`, `validate_regex`.
- Tests: `test_smtp` (returns the result, also on failure), `test_tg_bot`, `test_discord`.

`update` replaces every setting, so start from `get_all()`. Secrets come back blank and a blank secret keeps the stored value; pass `clear_*` to remove one.

API tokens: `get_api_tokens`, `add_api_token` (the only call that returns the token value), `delete_api_token` and `set_api_token_enabled`. The last two need the stored scope of the token in `expected_scope`; a mismatch is rejected by the panel.
