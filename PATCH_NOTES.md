# Patch notes

## Since b51ece4 (`docs: refresh installation guide and download source`)

- Replace the Rust installer’s comma separated extra-port prompt with an additional-port count. Generate consecutive ports after the primary port and reject counts that would exceed 65535.
- Update the installation guide with a concrete example: primary port 10223 plus four extra ports produces 10224 through 10227.

Validation: `bash -n shadowsocks.sh`, `git diff --check`, and the example port calculation passed. A live install was not run.

## Since 6af94a4 (`feat: add shadowsocks-rust multiport and network tuning`)

- Rewrite the installation guide with the current download URL, interactive choices, Rust multiport setup, service management, BBR checks, and uninstall instructions.
- Point the installer’s helper-file downloads to the same repository as the installation guide so the new Rust service and sysctl files are available.

Validation: `bash -n shadowsocks.sh`, `git diff --check`, and an HTTP 200 check for the documented script URL passed. A live install was not run.

## Since e5d5423 (`fix: debian support for libpcre`)

- Add comma separated Shadowsocks-Rust ports during installation. The installer writes a multi-server configuration and opens each port in the supported CentOS firewall path.
- Add a systemd instance template for running separate `ssserver` processes with their own configuration files.
- Add a managed sysctl profile for connection queues and local port range. Enable BBR with `fq` when the running kernel provides BBR.
- Raise the SysV service file descriptor limit; retain the existing systemd `LimitNOFILE=1048576` setting for the main service and apply it to instances.
- Document setup, client-side distribution across ports, verification, and cleanup in the README.

Validation: `bash -n shadowsocks.sh`, `sh -n scripts/shadowsocks-rust`, and `git diff --check` passed. A live install was not run in this workspace.
