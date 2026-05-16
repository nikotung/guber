# Guber - Nacos Application-IP-Hosts Register

Guber is a tool designed to automatically sync application IP addresses from Nacos to your local `/etc/hosts` file. This is particularly useful in dynamic environments like Kubernetes where application IPs change frequently, allowing you to access services directly by hostname while bypassing gateway authentication.

## Project Overview

- **Purpose**: Automates local hostname resolution for services registered in Nacos.
- **Key Features**:
    - Syncs Nacos service IPs to local hosts.
    - Supports multiple environments (e.g., dev, test, prod).
    - Periodic background updates (every 10 seconds).
    - Metadata-based filtering of service instances.
    - Automatic backup and restoration of the hosts file.
    - Can run as a standalone CLI or as a system service (via `kardianos/service`).
- **Core Technologies**:
    - **Language**: Go 1.21+
    - **CLI Framework**: [Cobra](https://github.com/spf13/cobra)
    - **Configuration**: [Viper](https://github.com/spf13/viper)
    - **Hosts Management**: [Txeh](https://github.com/txn2/txeh)
    - **Logging**: [Zap](https://go.uber.org/zap)
    - **Nacos Integration**: Custom HTTP-based Nacos client implementation.

## Getting Started

### Prerequisites

- Go 1.21 or later.
- Access to a Nacos server.
- Superuser/Admin privileges (required for modifying the hosts file).

### Configuration

Create a YAML configuration file (default `config.yaml`):

```yaml
log:
  level: info
service:
  - names:
      - "my-service-a"
      - "my-service-b"
    env: dev
    nacos:
      addr: "http://nacos.dev.local:8848"
      username: "nacos"
      password: "nacos"
      namespaceId: "public"    # Optional
      groupName: "DEFAULT_GROUP" # Optional
      clusterName: "DEFAULT"     # Optional
    keep:
      # Optional: only keep instances with specific metadata
      - key: "version"
        value: "1.0.0"
```

### Commands

- **Run in foreground**:
  ```bash
  sudo ./guber start -c config.yaml
  ```
- **Dry run (test configuration)**:
  ```bash
  ./guber run -c config.yaml
  ```
- **Service Management**:
  ```bash
  sudo ./guber service install -c $(pwd)/config.yaml
  sudo ./guber service start
  sudo ./guber service status
  sudo ./guber service stop
  sudo ./guber service uninstall
  ```

## Development

### Building

Use the provided build script for cross-compilation:

```bash
./build.sh
```

Binaries will be generated in the `bin/` directory for Darwin, Linux, and Windows.

### Project Structure

- `main.go`: Entry point, registers the `version` command.
- `cmd/`: Core logic and CLI commands.
    - `cli.go`: Defines the root and `start`/`run` commands.
    - `service.go`: System service management logic.
    - `guber.go`: Main `Guber` struct, handles hosts file updates and background watching.
    - `nacos.go`: Nacos API client implementation (auth and service discovery).
    - `config.go`: Configuration structures.

### Conventions

- **Safe Shutdown**: Uses `github.com/IrineSistiana/mosdns/v5/pkg/safe_close` for orchestrating graceful shutdowns and cleanup (restoring hosts file).
- **Logging**: Uses `zap` via a thin wrapper in `mlog`. Prefer `mlog.L()` for logging.
- **Hosts File Safety**: Always creates a `.bak` backup before modification and restores it on clean exit.
- **Error Handling**: Silent usage/errors in CLI are preferred; detailed errors are logged via `mlog`.

## Roadmap / Missing Features

- **Testing**: Currently, the project lacks automated tests. Adding unit tests for Nacos client logic and hosts manipulation would be a priority.
- **Windows Support**: While cross-compiled, service management and hosts file paths on Windows should be verified.
- **Nacos SDK**: Consider moving from custom HTTP calls to the official Nacos Go SDK if more complex features are needed.
