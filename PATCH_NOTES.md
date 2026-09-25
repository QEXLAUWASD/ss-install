# Patch notes

## Since e5d5423 (`fix: debian support for libpcre`)

- Add comma separated Shadowsocks-Rust ports during installation. The installer writes a multi-server configuration and opens each port in the supported CentOS firewall path.
- Add a systemd instance template for running separate `ssserver` processes with their own configuration files.
- Add a managed sysctl profile for connection queues and local port range. Enable BBR with `fq` when the running kernel provides BBR.
- Raise the SysV service file descriptor limit; retain the existing systemd `LimitNOFILE=1048576` setting for the main service and apply it to instances.
- Document setup, client-side distribution across ports, verification, and cleanup in the README.

Validation: `bash -n shadowsocks.sh`, `sh -n scripts/shadowsocks-rust`, and `git diff --check` passed. A live install was not run in this workspace.
