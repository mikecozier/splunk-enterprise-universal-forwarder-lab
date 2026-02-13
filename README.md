# splunk-enterprise-universal-forwarder-lab
---

## Recommended repo structure

```text
splunk-enterprise-universal-forwarder-lab/
├── README.md
├── architecture/
│   └── splunk-architecture.png
├── indexer/
│   ├── inputs.conf
│   └── indexes.conf
├── forwarder/
│   ├── inputs.conf
│   └── outputs.conf
├── screenshots/
│   ├── splunk-search-linux-test.png
│   ├── forwarder-status.png
│   └── listener-9997.png
└── notes/
    └── troubleshooting.md
---

Splunk Enterprise + Universal Forwarder Linux Log Pipeline

Overview

This project demonstrates a working Splunk Enterprise deployment ingesting Linux system logs from a remote host using the Splunk Universal Forwarder. The setup mirrors a real-world SOC / SIEM ingestion pipeline.

---

Architecture

![Image](https://www.splunk.com/content/dam/splunk-blogs/images/2016/12/Canvas-1-1.png)

![Image](https://splunk.deploy.heretto.com/v4/deployments/lbx3FHoDR4kUISPo5g64/object/333d76ea-49db-445f-adec-efc6a8f78028?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJodHRwczovL2pvcnNlay5jb20vZXpkX29yZ2FuaXphdGlvbiI6InNwbHVuayIsImh0dHBzOi8vam9yc2VrLmNvbS9lemQvb2JqZWN0X3V1aWQiOiIzMzNkNzZlYS00OWRiLTQ0NWYtYWRlYy1lZmM2YThmNzgwMjgiLCJleHAiOjE3NjU5NDUxNTIsImp0aSI6IjEyY2IxOGIxZWI1MjQ2MDE5ZWMxYjM2YmJlMDE0YTlkIiwiaHR0cHM6Ly9qb3JzZWsuY29tL2V6ZF9maWxlc2V0IjoieUw3Q2szSjF6U3JJVG9ocWdoS0EifQ.dV60naifyeSNAYp24Gc6EFFT5_Hh_zJCi5OmqMjO-0c)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AqFoKq2v9xEHjhYegY9ZWbA.png)

```
[ Linux Host: mainserver ]
  - /var/log/syslog
  - /var/log/auth.log
        |
        | (Splunk Universal Forwarder)
        |  TCP 9997
        v
[ Splunk Enterprise VM ]
  - Index: linux_test
  - Search Head + Indexer
```

---

Environment

| Component           | Details                        |
| ------------------- | ------------------------------ |
| Splunk Enterprise   | Installed on Ubuntu 24.04      |
| Universal Forwarder | Installed on Ubuntu Linux host |
| Forwarded Logs      | syslog, auth.log               |
| Index               | linux_test                     |
| Listener Port       | 9997                           |

---

Data Ingestion Flow

1. Universal Forwarder monitors Linux log files
2. Logs are forwarded to Splunk Enterprise over TCP 9997
3. Data is indexed into `linux_test`
4. Events are searchable by `host`, `source`, and `sourcetype`

---

Example Searches

```spl
index=linux_test
```

```spl
index=linux_test host=mainserver
```

```spl
index=linux_test sourcetype=syslog
```

```spl
| stats count by host source
```

---

### Configuration Highlights

**Indexer – TCP Input**

```ini
[splunktcp://9997]
disabled = 0
index = linux_test
```

**Forwarder – Monitored Files**

```ini
[monitor:///var/log/syslog]
index = linux_test

[monitor:///var/log/auth.log]
index = linux_test
```

---

### Validation

* Confirmed active forwarder connection
* Verified listener on port 9997
* Verified events stored at:

  ```
  /opt/splunk/var/lib/splunk/linux_test/
  ```

---

### Troubleshooting & Lessons Learned

* Resolved Linux log permission issues by adding UF user to `adm`
* Learned index vs host distinction
* Verified ingestion using `_internal` and filesystem checks

---

### Next Steps

* Add dashboards for SSH activity
* Create alerts for failed logins
* Expand ingestion to additional Linux hosts

---

## Screenshots (high impact)

Add **3 screenshots** (you already have them):
