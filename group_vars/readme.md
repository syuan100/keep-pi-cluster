(not up to date)

#### Deployment variables

| Variable | Type | Definition |
|--|--|--|
| `deploy_lighthouse_beacon` | `boolean` (`true`/`false`) | Deploy a lighthouse beacon |

#### Raspberry Pi Setup

| Variable | Type | Definition |
|--|--|--|
| `ansible_user` | `string` | Default user on Rapsberry Pis (will be `ubuntu` for Ubuntu OS) |
| `copy_local_key` | `string` | Path to local key on host machine (defaults to `$HOME/.ssh/id_rsa.pub`) |
| `sys_packages` | `Array[] string` | List of default packages to install |

#### Kubernetes setup

| Variable | Type | Definition |
|--|--|--|
| `k3s_version` | `string` | k3s version to install |
| `systemd_dir` | `string` | Path to systemd directory |
| `master_ip` | `string` | Valid IP address for master host (Automatically pulled from hosts.ini file) |
| `extra_k3s_server_args` | `string` | Additional arguments for `k3s server` command (Default is `--no-deploy servicelb` since we are using MetalLB as the load balancer) |

#### Replicated Storage

| Variable | Type | Definition |
|--|--|--|
| `storage_solution` | `string` | Storage solution to use (Currently only `glusterfs`) |
| `storage_path` | `string` | Path to disk to use for replicated storage |
| `storage_size` | `int` | Amount in GB to use for each disk |

#### Docker images

| Variable | Type | Definition |
|--|--|--|
| `geth_image_tag` | `string` | Valid container tag/path for geth |
| `lighthouse_image_tag` | `string` | Valid container tag/path for lighthouse |

*NOTE: Images must be built for arm64 architectures in order to deploy correctly*