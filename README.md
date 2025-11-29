# npm packages security best practices for developers


## 1. **Package Legitimacy**

 **Verify the source**

- [ ] Always install from the official [npm registry](https://www.npmjs.com/), not external URLs or Git repositories.\
  Avoid dependencies like:

  ```json
    "dependency-name": "http://example.com/npm/pkg"
  ```
  
- [ ] Confirm the package has a consistent publisher (same maintainer for versions).

**Check the maintainer**

- [ ] Review the maintainer’s npm profile:
  - Joined date
  - Number of maintained packages
  - Past publication history
- [ ] Be cautious if the publisher is newly created or renamed recently.

**Check for typosquatting**

- [ ] Compare the package name against popular legitimate ones (`react`, `lodash`, `core-js`, etc.).\
  Attackers often publish similar names, e.g. `corejs`, `polyfill-corejs`, `lodashs`.

**Check the download pattern**

- [ ] Review weekly/monthly download trends on npmjs.com.
  - A sudden spike or huge drop could indicate malicious republishing.
  - Example red flag: 0 downloads for months, then sudden thousands overnight.

---

## 2. **Package Contents Inspection**

 **Review**

- [ ] Check for suspicious **scripts**:

  ```json
  "scripts": {
    "preinstall": "...",
    "install": "...",
    "postinstall": "..."
  }
  ```

  These run automatically during `npm install` — often abused for malware.\
  Red flag: `curl`, `wget`, `node-fetch`, or encoded payloads.

- [ ] Verify `dependencies` and `devDependencies`:

  - Avoid dependencies pointing to external HTTP/HTTPS sources.
  - Avoid dependencies using Git URLs or IP addresses.

**Audit entry points**

- [ ] Make sure entry points (`main`, `bin`) refer to legitimate project files, not hidden or obfuscated scripts.

**Inspect package size**

- [ ] If a small library suddenly becomes huge (megabytes of content), investigate with:
  ```bash
  npm pack --dry-run
  ```
  and inspect the tarball contents.

**Look for obfuscation**

- [ ] Red flags: unreadable variable names, Base64 strings, unusual eval/Function calls, or `require()` with computed paths.

**Static analysis**

- Use tools like:
  - [npm audit](https://docs.npmjs.com/cli/v9/commands/npm-audit)
  - [socket.dev](https://socket.dev/)
  - [snyk.io](https://snyk.io/)
  - [Nodejitsu NSP](https://nodesecurity.io/)

---

## 3. **Dependency Graph Review**

**Check transitive dependencies**

- [ ] Run:
  ```bash
  npm ls --all
  ```
  and verify the full dependency tree.

**Lock your dependencies**

- [ ] Always use `package-lock.json` or `npm ci` to ensure reproducible builds.
- [ ] Verify integrity hashes (`integrity` field in lockfile).

**Pin exact versions**

- [ ] Avoid version ranges like `^1.2.3` or `~1.2.3`.\
  Instead use exact:
  ```json
  "dependency": "1.2.3"
  ```

**Disallow install-time scripts globally**

- [ ] Use:
  ```bash
  npm install --ignore-scripts
  ```
  Then manually enable trusted ones.

---

## 4. **Code & Repository Integrity** 

**Repository validation**

- [ ] Check the GitHub/GitLab link in `package.json`:
  - Does it exist?
  - Is the repository active?
  - Does the latest tag match the npm version?
- Watch for cloned repos that differ from the published npm code.

**Compare tarball and GitHub code**

- [ ] Run:
  ```bash
  npm pack <package-name>
  ```
  and manually compare to the upstream repo’s code.

**Verify release provenance**

- [ ] Legitimate projects often sign releases, use GitHub Actions for publishing, or tag releases consistently.

**Look for hidden or duplicate packages**

- [ ] Search npm for similar names from the same author — if several nearly identical packages exist with minor differences, treat as suspicious.

---

## 5. **Runtime Behavior**

**Monitor network access**

- [ ] If a package initiates network calls during `install` or runtime, flag for investigation.\
  Run installs in a sandboxed environment:
  ```bash
  npm install --ignore-scripts
  strace -e trace=network node ...
  ```

**Check system interactions**

- [ ] Review for calls to:
  - `fs.*` (file system)
  - `child_process.exec`
  - `net.*`, `https.*`, or `fetch`
  - External IPs (e.g. `ipify.org`, `storeartifact.com`)

**Run dynamic sandbox analysis**

- [ ] Use containers or VMs for initial installs.\
  Tools like `nodejsscan` or `Falco` can detect abnormal behaviors.

---

## 6. **Community & Reputation**

**Check issue tracker**

- [ ] Is it active? Are there reports of malware or weird behavior?

**Check for recent takeover events**

- [ ] If ownership changed recently (new maintainer or new version after long silence), treat cautiously.

**Read changelogs**

- [ ] Large unexplained changes, especially adding install scripts, are major red flags.

**Check digital footprint**

- [ ] Search security feeds (Sonatype, ReversingLabs, Socket.dev, Checkmarx, etc.) for mentions of the package.

---

## 7. **Enterprise / CI/CD Controls**

**Use an internal npm mirror**

- [ ] Proxy through a vetted internal registry (e.g., Artifactory, Verdaccio, Nexus).\
  This allows caching and review before internal use.

**Enable dependency allowlists**

- [ ] Define which packages or maintainers are trusted for production.

**Implement continuous dependency scanning**

- [ ] Integrate scanners into CI (Snyk, Dependabot, Socket.dev, OWASP Dependency Track).

**Use SBOMs (Software Bill of Materials)**

- [ ] Generate SBOMs (`cyclonedx-npm`, `syft`) to track dependencies over time.

**Isolate build environments**

- [ ] Never install dependencies on CI machines that share credentials.
- [ ] Use ephemeral runners with restricted access.

---

## 8. **Incident Response & Remediation**

**Have a rollback plan**

- [ ] Maintain reproducible builds so you can pin to last known safe versions.

**Revoke and rotate credentials**

- [ ] If you installed a malicious package, assume credential compromise.

**Notify affected users**

- [ ] If your package consumed a malicious dependency, disclose quickly and transparently.

**Report to npm security team**

- [ ] Email: [security@npmjs.com](mailto\:security@npmjs.com) or submit via the [npm Advisory Form]([https://www.npmjs.com/advisories/report](https://docs.npmjs.com/reporting-malware-in-an-npm-package)).

---

