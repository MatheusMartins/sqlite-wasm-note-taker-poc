Proof of concept of SQLite via WebAssembly module implemented on a note taking app.

- **SQLite WASM**: Full SQL database engine running client-side via `sql.js`.
- **Persistence**: Saves the database state to the browser's Origin Private File System.
- **Note Management**: Simple interface to create, view, and reset a notes database.


## How to Run

This application requires specific MIME types (`application/wasm`) to be served correctly by the browser. Opening the `index.html` file directly from your file system will likely fail.

Use the following Docker command to start a local server that handles these headers correctly:

```bash
docker run --rm -it -p 3331:8080 -v "$PWD":/www busybox \
  sh -c "echo -e '# httpd.conf\n.wasm:application/wasm\n.js:application/javascript\n.html:text/html' > /etc/httpd.conf && httpd -f -p 8080 -h /www"
```


## Known Issues

- Cross-Site Scripting (XSS)

  Issue: inserting database values directly into the DOM without sanitization (e.g. <script>alert('XSS')</script>).

  Impact: data stolen, page manipulation, perform actions on behalf of the user.

  Solutions: Use a sanitization library (e.g. DOMpurify) or use a reactive library like Svelte. 

- Content Security Policy (CSP)

  Issue: No CSP headers or meta tags restrict what resources can load or what scripts can execute.

  Impact: Makes XSS exploitation easier and allows inline scripts to run without restriction.

  Solution: Serve this website from a server you control and configure strict CSP - example: `Content-Security-Policy: default-src 'self'; script-src 'self' https://cdnjs.cloudflare.com;`

- Data Exposure in Browser

  Issue: No encryption at rest, sensitive notes are stored unencrypted.

  Impact: Anyone with access to the computer can access the notes.

  Solution: Store keys in memory and prompt it from the user on every load. Storing keys on storage will defeat the purpose (indexedDB for example would still be accessible via the file system).

- Main thread usage

  Issue: The app relies on the main thread for its operations.

  Impact: Can lead to UI freezes and unresponsiveness during database operations.

  Solution: Offload database operations to a Web Worker to keep the UI responsive.

