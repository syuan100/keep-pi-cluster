(not up to date)

#### Deployment variables

*NOTE: It is recommended to only deploy geth on Pis with 4GB of RAM or more*

| Variable | Type | Definition |
|--|--|--|
| `deploy_geth` | `boolean` (`true`/`false`) | Deploy a local geth instance (default: false) |
| `deploy_keep_random_beacon` | `boolean` (`true`/`false`) | Deploy a KEEP random beacon staking node (default: true) |
| `deploy_keep_ecdsa` | `boolean` (`true`/`false`) | Deploy a KEEP ECDSA staking node (default: true) |
| `external_eth1` | `boolean` (`true`/`false`) | Set to true if you are using an external Ethereum endpoint like Infura |
| `eth1_endpoint` | `string` | URL for Ethereum RPC endpoint |
 
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

*NOTE: Images must be built for arm64 architectures in order to deploy correctly*

| Variable | Type | Definition |
|--|--|--|
| `keep_random_beacon_tag` | `string` | Valid container tag/path for KEEP random beacon node |
| `keep_ecdsa_tag` | `string` | Valid container tag/path for KEEP ECDSA node |

#### Geth variables

*NOTE: It is recommended to only deploy geth locally on Pis with 4GB of RAM or more*

| Variable | Type | Definition |
|--|--|--|
| `geth_json_rpc_port` | `int` | geth container JSON RPC port (defaults to 8545) |
| `geth_json_rpc_port_target` | `int` | geth service JSON RPC port (defaults to 8545) |
| `geth_port` | `int` | Main geth container port (defaults to 30303) |
| `geth_port_target` | `int` | Main geth service port (defaults to 30303) |

#### Keep variables

| Variable | Type | Definition |
|--|--|--|
| `container_mount_point` | `string` | Internal container mount point for Docker images (default: /mnt)  |
| `ethereum_url` | `string` | wss:// endpoint for Ethereum endpoint |
| `beacon_account_address` | `string` | Ethereum address for your account that is authorized for random beacon staking |
| `beacon_keystore_filename` | `string` | Exact filename of ethereum account keystore file |
| `beacon_account_keystore` | `string` | Full path to keystore file on Kubernetes master |
| `beacon_account_password` | `string` | Ethereum account password |
| `beacon_p2p_port` | `int` | Main beacon p2p port (defaults to 3919) |
| `beacon_announced_addresses` | `string` | Comma separated list of announced p2p adddresses for your node |
| `beacon_debug_level` | `string` | Log level for beacon node |
| `ecdsa_account_address` | `string` | Ethereum address for your account that is authorized for ECDSA staking |
| `ecdsa_keystore_filename` | `string` | Exact filename of ethereum account keystore file (ECDSA) |
| `ecdsa_account_keystore` | `string` | Full path to keystore file on Kubernetes master (ECDSA) |
| `ecdsa_account_password` | `string` | Ethereum account password (ECDSA) |
| `ecdsa_p2p_port` | `int` | Main ECDSA p2p port (defaults to 3920) |
| `ecdsa_announced_addresses` | `string` | Comma separated list of announced p2p adddresses for your node (ECDSA) |
| `ecdsa_debug_level` | `string` | Log level for ECDSA node |


