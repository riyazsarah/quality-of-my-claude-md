# Secret Detection Patterns

Use during Phase 2 (Static Analysis) to flag potential secrets or credentials found in context files.

**Important:** These are regex-based heuristics. False positives are expected and acceptable — the user reviews all findings before any action is taken.

## Patterns

### SEC-001: AWS Access Key ID

- **Regex:** `AKIA[0-9A-Z]{16}`
- **Description:** AWS IAM access key IDs always start with `AKIA` followed by 16 alphanumeric characters
- **Example match:** `AKIAIOSFODNN7EXAMPLE`
- **False positive notes:** Very low — the AKIA prefix is AWS-specific

### SEC-002: AWS Secret Access Key

- **Regex:** `(?i)aws_secret_access_key\s*[=:]\s*[A-Za-z0-9/+=]{40}`
- **Description:** AWS secret keys are 40-character base64 strings, usually preceded by a key name
- **Example match:** `aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
- **False positive notes:** Medium — the key name prefix reduces false positives

### SEC-003: Generic API Key

- **Regex:** `(?i)(api[_-]?key|apikey)\s*[=:]\s*['"]?[A-Za-z0-9_\-]{20,}['"]?`
- **Description:** Generic API key assignments in various formats
- **Example match:** `api_key = "sk_live_abc123def456ghi789"`
- **False positive notes:** Medium — may match example/placeholder keys

### SEC-004: OpenAI / Anthropic API Key

- **Regex:** `(sk-[a-zA-Z0-9]{20,}|sk-ant-[a-zA-Z0-9\-]{20,})`
- **Description:** OpenAI keys start with `sk-`, Anthropic keys start with `sk-ant-`
- **Example match:** `sk-proj-abc123def456ghi789jkl012mno345`
- **False positive notes:** Low — prefix is provider-specific

### SEC-005: GitHub Token

- **Regex:** `(ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|ghu_[a-zA-Z0-9]{36}|ghs_[a-zA-Z0-9]{36}|ghr_[a-zA-Z0-9]{36}|github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59})`
- **Description:** GitHub personal access tokens, OAuth tokens, and fine-grained tokens
- **Example match:** `ghp_ABCDEFghijklmnopqrstuvwxyz1234567890`
- **False positive notes:** Very low — prefixes are GitHub-specific

### SEC-006: Private Key Block

- **Regex:** `-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----`
- **Description:** PEM-encoded private key headers
- **Example match:** `-----BEGIN RSA PRIVATE KEY-----`
- **False positive notes:** Very low — this is a standard key format header

### SEC-007: Generic Password Assignment

- **Regex:** `(?i)(password|passwd|pwd)\s*[=:]\s*['"]?[^\s'"]{8,}['"]?`
- **Description:** Password assignments in config-like patterns
- **Example match:** `password = "MySecr3tP@ss!"`
- **False positive notes:** High — may match documentation about password requirements or placeholders

### SEC-008: Connection String

- **Regex:** `(?i)(mongodb(\+srv)?|postgres(ql)?|mysql|redis|amqp)://[^\s'"]+:[^\s'"]+@[^\s'"]+`
- **Description:** Database/service connection strings with embedded credentials
- **Example match:** `mongodb+srv://admin:password123@cluster.mongodb.net/db`
- **False positive notes:** Low — the URL format with credentials is distinctive

### SEC-009: Bearer / Authorization Token

- **Regex:** `(?i)(bearer|authorization)\s*[=:]\s*['"]?[A-Za-z0-9._\-]{20,}['"]?`
- **Description:** Bearer tokens or authorization headers with token values
- **Example match:** `Authorization: Bearer eyJhbGciOiJIUzI1NiIs...`
- **False positive notes:** Medium — may match example documentation

### SEC-010: Slack / Discord Webhook URL

- **Regex:** `https://hooks\.slack\.com/services/T[A-Z0-9]+/B[A-Z0-9]+/[a-zA-Z0-9]+` and `https://discord(app)?\.com/api/webhooks/[0-9]+/[A-Za-z0-9_\-]+`
- **Description:** Webhook URLs that could be used to send unauthorized messages
- **Example match:** Slack: `hooks.slack.com/services/T.../B.../...` | Discord: `discord.com/api/webhooks/.../...`
- **False positive notes:** Very low — URL format is platform-specific

## Usage Notes

1. Run each pattern against every line of every discovered context file
2. When a match is found, report:
   - Pattern ID and name
   - File path and line number
   - The matched text (partially redacted — show first 4 and last 4 characters only)
   - Confidence based on the pattern's false positive rating
3. Group all secret findings in a dedicated "Security Warnings" section of the output
4. Secret findings do NOT affect the 6-dimension quality score — they are reported separately
5. If 3+ secrets are found in a single file, add a prominent warning at the top of the report
