# Credential pivot — `10.20.160.102` (Linux — not **JUPITER**-joined)

**Secrets:** only in [`../credentials_handoff_SECRETS.md`](../credentials_handoff_SECRETS.md) (gitignored).

**Prep:** [Credential pivot hub](../credential_pivot_A_Rocha_hosts.md).

---

## Tomcat manager (XENA cred reuse — usually fails on LAMPP; fast check)

Set **`U`** / **`P`** from **XENA** **`tomcat`** / **`manager`** rows in the secrets file.

```bash
export H102=10.20.160.102
export U='tomcat'    # or manager / rob — from handoff
export P='<FROM_SECRETS_FILE>'
curl -sI -u "$U:$P" "http://$H102:8080/manager/html"
curl -sI -u "$U:$P" "https://$H102:8443/manager/html"
```

## MySQL on `.102` (only if `nmap` shows **3306** listening externally)

```bash
nmap -Pn -p3306 -sV "$H102"
mysql --protocol=TCP -h "$H102" -P 3306 -u root -p
```

Prefer creds from **your FTP files** on **.101** or **RFI**-readable configs—not random team passwords—unless you confirmed reuse.

## SSH (only if port **22** open)

```bash
nmap -Pn -p22 -sV "$H102"
# ssh user@$H102   # only if a teammate gave a Linux reuse password
```
