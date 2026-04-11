# DDNS Sync

A Jenkins pipeline that runs regularly to keep a Cloudflare DNS A record in sync with the server's current public IP address. If the IP has changed, it updates the record via the Cloudflare API. An email alert is sent in case of failure.

## Requirements

- Jenkins with a `docker` agent
- A Jenkins secret file credential named `ddns-sync.env` containing:

```env
API_TOKEN=        # Cloudflare API token with DNS edit permissions
ZONE_ID=          # Cloudflare Zone ID for the domain
RECORD_ID=        # Cloudflare DNS Record ID to update
DOMAIN=           # Fully qualified domain name of the A record (e.g. home.example.com)
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
