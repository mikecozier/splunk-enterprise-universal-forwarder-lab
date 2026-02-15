# splunk-enterprise-universal-forwarder-lab

Enterprise-style Splunk log ingestion pipeline using Splunk Enterprise and Universal Forwarder to collect Linux system logs over TCP 9997.

---

## Overview

This lab demonstrates a production-style Splunk ingestion architecture where a Linux host forwards system logs to a centralized Splunk Enterprise instance for indexing and search.

The pipeline simulates a real-world SOC / SIEM deployment:

* Remote log collection
* Secure TCP forwarding
* Custom index configuration
* Sourcetype normalization
* End-to-end validation and troubleshooting

---

## Architecture

```
[ Linux Host: mainserver ]
  - /var/log/syslog
  - /var/log/auth.log
        |
        |  Splunk Universal Forwarder
        |  TCP 9997 (Splunk-to-Splunk)
        v
[ Splunk Enterprise VM ]
  - TCP Listener (splunktcp://9997)
  - Custom Index: linux_test
  - Search Head + Indexer
```

---

## Environment

| Component           | Details                    |
| ------------------- | -------------------------- |
| OS (Indexer)        | Ubuntu 24.04 LTS           |
| OS (Forwarder)      | Ubuntu Linux               |
| Splunk Enterprise   | Single-instance deployment |
| Universal Forwarder | Remote Linux host          |
| Logs Forwarded      | syslog, auth.log           |
| Custom Index        | linux_test                 |
| Listener Port       | 9997                       |

---

## Data Ingestion Flow

1. Universal Forwarder monitors:

   * `/var/log/syslog`
   * `/var/log/auth.log`

2. Events are forwarded via:

   ```
   tcpout → 192.168.x.x:9997
   ```

3. Splunk Enterprise listens on:

   ```ini
   [splunktcp://9997]
   disabled = 0
   index = linux_test
   ```

4. Events are indexed into:

   ```
   $SPLUNK_HOME/var/lib/splunk/linux_test/
   ```

5. Events become searchable via SPL.

---

## Configuration Highlights

### Indexer Configuration

**Listener Configuration**

```ini
[splunktcp://9997]
disabled = 0
index = linux_test
```

**Custom Index**

```ini
[linux_test]
homePath   = $SPLUNK_DB/linux_test/db
coldPath   = $SPLUNK_DB/linux_test/colddb
thawedPath = $SPLUNK_DB/linux_test/thaweddb
```

---

### Universal Forwarder Configuration

**Monitored Files**

```ini
[monitor:///var/log/syslog]
disabled = 0
index = linux_test
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = 0
index = linux_test
sourcetype = linux_secure
```

**Forwarding Target**

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.x.x:9997
```

---

## Example Searches

Basic ingestion check:

```spl
index=linux_test
```

Host validation:

```spl
index=linux_test host=mainserver
```

By sourcetype:

```spl
index=linux_test sourcetype=syslog
```

Aggregation example:

```spl
| stats count by host source sourcetype
```

---

## Validation Performed

* Verified active forwarder connection:

  ```
  splunk list forward-server
  ```

* Confirmed TCP listener on indexer:

  ```
  ss -lntp | grep 9997
  ```

* Generated test event:

  ```
  logger "splunk_test_pipeline"
  ```

* Confirmed ingestion via SPL search

* Verified raw bucket storage on filesystem:

  ```
  /opt/splunk/var/lib/splunk/linux_test/
  ```

---

## Troubleshooting & Lessons Learned

* Resolved Linux permission issue by adding UF user to `adm` group
* Used `btool --debug` to validate active configuration
* Distinguished between:

  * `host`
  * `source`
  * `sourcetype`
  * `index`
* Verified end-to-end ingestion using both filesystem and SPL queries
* Cleaned up configuration placement (`system/local` vs app directories)

---

## Screenshots

Include the following:

* Splunk search results showing indexed events
* Forwarder status (active connection)
* TCP listener on port 9997

These screenshots demonstrate operational validation of the ingestion pipeline.

---

## Next Steps

* Create dashboard for SSH authentication activity
* Build alert for repeated failed logins
* Enable SSL encryption between forwarder and indexer
* Add additional Linux hosts for horizontal scaling
* Implement index retention policies

---

## Why This Project Matters

This lab demonstrates hands-on experience with:

* SIEM log ingestion pipelines
* Splunk index management
* Forwarder-to-indexer architecture
* Linux log collection
* Troubleshooting distributed logging systems
* Production-style validation methods

This setup mirrors real-world enterprise Splunk deployments used in SOC and security monitoring environments.

---
