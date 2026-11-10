## OnlyOffice DocumentServer whitelist

### Versioned assets
When a document is opened, the DocumentServer loads its whole front end from a
versioned path: `/<version>-<hash>/...`, where the hash is the build hash of the
installed DocumentServer.

Most of those files have an extension (`.js`, `.css`, `.png`, `.wasm`) and are
detected as static assets, but the fonts and some cache/download endpoints do
not: `/10.0.0-5e5ceea51d7aa3e6c776c4125a30031c/fonts/078` is one request per
font, each with a different file name. Opening a document can therefore produce
more than 40 distinct non-static file names in 20 seconds, which triggers
`crowdsecurity/http-crawl-non_statics` and, with the default remediation, a 168
hour ban for a user who only opened a file.

The whitelist exempts `GET` requests under the versioned prefix that answer
`200` or `304` (the statuses these assets return in practice). The prefix is
anchored, so a URL that only contains it in the query string is not exempt, and
the optional `/onlyoffice` segment covers DocumentServer installations published
under a subpath. Websocket upgrades (`101`, `/doc/<id>/c/`) are left out.
