# Web

You can create `$HOME/public_html` and Apache http exposes content to `/~$USER/`.

Content negotiation is enabled by default, so a request to `/~foo/bar` serves `/home/foo/public_html/bar.html`.
Content negotiation applies to indexes too.
