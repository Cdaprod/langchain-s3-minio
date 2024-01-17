# Langchain Community S3 Directory Loader  

What were doing with Langchain, MinIO, and OpenAI


1. Load the bucket contents with `S3 Directory Loader`
2. Load a file with `S3 File Loader`
3. Summarize `S3 File Loader` with OpenAI
4. Summarize `S3 Directory Loader` with OpenAI

Resources were accessing:
- Endpoint: https://play.min.io
- Bucket: "web-documentation"

Bucket contains files:
- `minio_quickstart.md`
- `test-file-1.md`
- `test-file-2.md`

We'll break down the process into two distinct parts: one for `S3DirectoryLoader` and the other for `S3FileLoader`. Each part will have its own set of code blocks and explanations. Let's start by detailing the steps and code for each loader.

---

---

Install Langchain with `pip install langchain`.


```python
pip install langchain
```

    Requirement already satisfied: langchain in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (0.0.352)
    Requirement already satisfied: PyYAML>=5.3 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (6.0)
    Requirement already satisfied: SQLAlchemy<3,>=1.4 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (2.0.16)
    Requirement already satisfied: aiohttp<4.0.0,>=3.8.3 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (3.8.4)
    Requirement already satisfied: async-timeout<5.0.0,>=4.0.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (4.0.2)
    Requirement already satisfied: dataclasses-json<0.7,>=0.5.7 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (0.5.8)
    Requirement already satisfied: jsonpatch<2.0,>=1.33 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (1.33)
    Requirement already satisfied: langchain-community<0.1,>=0.0.2 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (0.0.2)
    Requirement already satisfied: langchain-core<0.2,>=0.1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (0.1.0)
    Requirement already satisfied: langsmith<0.1.0,>=0.0.70 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (0.0.72)
    Requirement already satisfied: numpy<2,>=1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (1.23.3)
    Requirement already satisfied: pydantic<3,>=1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (1.10.12)
    Requirement already satisfied: requests<3,>=2 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (2.31.0)
    Requirement already satisfied: tenacity<9.0.0,>=8.1.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain) (8.2.3)
    Requirement already satisfied: attrs>=17.3.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (23.1.0)
    Requirement already satisfied: charset-normalizer<4.0,>=2.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (3.1.0)
    Requirement already satisfied: multidict<7.0,>=4.5 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (6.0.4)
    Requirement already satisfied: yarl<2.0,>=1.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (1.8.2)
    Requirement already satisfied: frozenlist>=1.1.1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (1.3.3)
    Requirement already satisfied: aiosignal>=1.1.2 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from aiohttp<4.0.0,>=3.8.3->langchain) (1.3.1)
    Requirement already satisfied: marshmallow<4.0.0,>=3.3.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from dataclasses-json<0.7,>=0.5.7->langchain) (3.19.0)
    Requirement already satisfied: marshmallow-enum<2.0.0,>=1.5.1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from dataclasses-json<0.7,>=0.5.7->langchain) (1.5.1)
    Requirement already satisfied: typing-inspect>=0.4.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from dataclasses-json<0.7,>=0.5.7->langchain) (0.9.0)
    Requirement already satisfied: jsonpointer>=1.9 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from jsonpatch<2.0,>=1.33->langchain) (2.3)
    Requirement already satisfied: anyio<5,>=3 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain-core<0.2,>=0.1->langchain) (3.7.1)
    Requirement already satisfied: packaging<24.0,>=23.2 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from langchain-core<0.2,>=0.1->langchain) (23.2)
    Requirement already satisfied: typing-extensions>=4.2.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from pydantic<3,>=1->langchain) (4.8.0)
    Requirement already satisfied: idna<4,>=2.5 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from requests<3,>=2->langchain) (2.10)
    Requirement already satisfied: urllib3<3,>=1.21.1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from requests<3,>=2->langchain) (1.26.15)
    Requirement already satisfied: certifi>=2017.4.17 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from requests<3,>=2->langchain) (2022.12.7)
    Requirement already satisfied: greenlet!=0.4.17 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from SQLAlchemy<3,>=1.4->langchain) (2.0.2)
    Requirement already satisfied: sniffio>=1.1 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from anyio<5,>=3->langchain-core<0.2,>=0.1->langchain) (1.3.0)
    Requirement already satisfied: exceptiongroup in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from anyio<5,>=3->langchain-core<0.2,>=0.1->langchain) (1.1.1)
    Requirement already satisfied: mypy-extensions>=0.3.0 in c:\users\david\appdata\local\packages\pythonsoftwarefoundation.python.3.9_qbz5n2kfra8p0\localcache\local-packages\python39\site-packages (from typing-inspect>=0.4.0->dataclasses-json<0.7,>=0.5.7->langchain) (1.0.0)
    Note: you may need to restart the kernel to use updated packages.


    DEPRECATION: torchsde 0.2.5 has a non-standard dependency specifier numpy>=1.19.*; python_version >= "3.7". pip 24.0 will enforce this behaviour change. A possible replacement is to upgrade to a newer version of torchsde or contact the author to suggest that they release a version with a conforming dependency specifiers. Discussion can be found at https://github.com/pypa/pip/issues/12063


## #1 - How to use the Langchain S3 Directory Loader

#### Objective
Load multiple documents from an S3 directory.

#### Steps
1. Import `S3DirectoryLoader` from `langchain_community.document_loaders`.
2. Configure MinIO credentials and endpoint.
3. Initialize `S3DirectoryLoader` with the specified bucket, directory prefix, endpoint URL, AWS access keys, and SSL usage.
4. Load documents from the specified directory.

#### Code Block


```python
from langchain_community.document_loaders.s3_directory import S3DirectoryLoader

# MinIO Configuration
endpoint = 'play.min.io:9000'
access_key = 'minioadmin'
secret_key = 'minioadmin'
use_ssl = True

# Initializing the S3DirectoryLoader
directory_loader = S3DirectoryLoader(
    bucket='web-documentation', 
    prefix='', 
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key, 
    aws_secret_access_key=secret_key, 
    use_ssl=use_ssl
)

# Load documents from directory
documents = directory_loader.load()
documents
```




    [Document(page_content='MinIO Quickstart Guide\n\nMinIO is a High Performance Object Storage released under GNU Affero General Public License v3.0. It is API compatible with Amazon S3 cloud storage service. Use MinIO to build high performance infrastructure for machine learning, analytics and application data workloads.\n\nThis README provides quickstart instructions on running MinIO on bare metal hardware, including container-based installations. For Kubernetes environments, use the MinIO Kubernetes Operator.\n\nContainer Installation\n\nUse the following commands to run a standalone MinIO server as a container.\n\nStandalone MinIO servers are best suited for early development and evaluation. Certain features such as versioning, object locking, and bucket replication\nrequire distributed deploying MinIO with Erasure Coding. For extended development and production, deploy MinIO with Erasure Coding enabled - specifically,\nwith a minimum of 4 drives per MinIO server. See MinIO Erasure Code Overview\nfor more complete documentation.\n\nStable\n\nRun the following command to run the latest stable image of MinIO as a container using an ephemeral data volume:\n\nsh\npodman run -p 9000:9000 -p 9001:9001 \\\n  quay.io/minio/minio server /data --console-address ":9001"\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded\nobject browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the\nroot credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See\nTest using MinIO Client mc for more information on using the mc commandline tool. For application developers,\nsee https://min.io/docs/minio/linux/developers/minio-drivers.html to view MinIO SDKs for supported languages.\n\nNOTE: To deploy MinIO on with persistent storage, you must map local persistent directories from the host OS to the container using the podman -v option. For example, -v /mnt/data:/data maps the host OS drive at /mnt/data to /data on the container.\n\nmacOS\n\nUse the following commands to run a standalone MinIO server on macOS.\n\nStandalone MinIO servers are best suited for early development and evaluation. Certain features such as versioning, object locking, and bucket replication require distributed deploying MinIO with Erasure Coding. For extended development and production, deploy MinIO with Erasure Coding enabled - specifically, with a minimum of 4 drives per MinIO server. See MinIO Erasure Code Overview for more complete documentation.\n\nHomebrew (recommended)\n\nRun the following command to install the latest stable MinIO package using Homebrew. Replace /data with the path to the drive or directory in which you want MinIO to store data.\n\nsh\nbrew install minio/stable/minio\nminio server /data\n\nNOTE: If you previously installed minio using brew install minio then it is recommended that you reinstall minio from minio/stable/minio official repo instead.\n\nsh\nbrew uninstall minio\nbrew install minio/stable/minio\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the root credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See Test using MinIO Client mc for more information on using the mc commandline tool. For application developers, see https://min.io/docs/minio/linux/developers/minio-drivers.html/ to view MinIO SDKs for supported languages.\n\nBinary Download\n\nUse the following command to download and run a standalone MinIO server on macOS. Replace /data with the path to the drive or directory in which you want MinIO to store data.\n\nsh\nwget https://dl.min.io/server/minio/release/darwin-amd64/minio\nchmod +x minio\n./minio server /data\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the root credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See Test using MinIO Client mc for more information on using the mc commandline tool. For application developers, see https://min.io/docs/minio/linux/developers/minio-drivers.html to view MinIO SDKs for supported languages.\n\nGNU/Linux\n\nUse the following command to run a standalone MinIO server on Linux hosts running 64-bit Intel/AMD architectures. Replace /data with the path to the drive or directory in which you want MinIO to store data.\n\nsh\nwget https://dl.min.io/server/minio/release/linux-amd64/minio\nchmod +x minio\n./minio server /data\n\nThe following table lists supported architectures. Replace the wget URL with the architecture for your Linux host.\n\n| Architecture                   | URL                                                        |\n| --------                       | ------                                                     |\n| 64-bit Intel/AMD               | https://dl.min.io/server/minio/release/linux-amd64/minio   |\n| 64-bit ARM                     | https://dl.min.io/server/minio/release/linux-arm64/minio   |\n| 64-bit PowerPC LE (ppc64le)    | https://dl.min.io/server/minio/release/linux-ppc64le/minio |\n| IBM Z-Series (S390X)           | https://dl.min.io/server/minio/release/linux-s390x/minio   |\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the root credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See Test using MinIO Client mc for more information on using the mc commandline tool. For application developers, see https://min.io/docs/minio/linux/developers/minio-drivers.html to view MinIO SDKs for supported languages.\n\nNOTE: Standalone MinIO servers are best suited for early development and evaluation. Certain features such as versioning, object locking, and bucket replication require distributed deploying MinIO with Erasure Coding. For extended development and production, deploy MinIO with Erasure Coding enabled - specifically, with a minimum of 4 drives per MinIO server. See MinIO Erasure Code Overview for more complete documentation.\n\nMicrosoft Windows\n\nTo run MinIO on 64-bit Windows hosts, download the MinIO executable from the following URL:\n\nsh\nhttps://dl.min.io/server/minio/release/windows-amd64/minio.exe\n\nUse the following command to run a standalone MinIO server on the Windows host. Replace D:\\ with the path to the drive or directory in which you want MinIO to store data. You must change the terminal or powershell directory to the location of the minio.exe executable, or add the path to that directory to the system $PATH:\n\nsh\nminio.exe server D:\\\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the root credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See Test using MinIO Client mc for more information on using the mc commandline tool. For application developers, see https://min.io/docs/minio/linux/developers/minio-drivers.html to view MinIO SDKs for supported languages.\n\nNOTE: Standalone MinIO servers are best suited for early development and evaluation. Certain features such as versioning, object locking, and bucket replication require distributed deploying MinIO with Erasure Coding. For extended development and production, deploy MinIO with Erasure Coding enabled - specifically, with a minimum of 4 drives per MinIO server. See MinIO Erasure Code Overview for more complete documentation.\n\nInstall from Source\n\nUse the following commands to compile and run a standalone MinIO server from source. Source installation is only intended for developers and advanced users. If you do not have a working Golang environment, please follow How to install Golang. Minimum version required is go1.19\n\nsh\ngo install github.com/minio/minio@latest\n\nThe MinIO deployment starts using default root credentials minioadmin:minioadmin. You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server. Point a web browser running on the host machine to http://127.0.0.1:9000 and log in with the root credentials. You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.\n\nYou can also connect using any S3-compatible tool, such as the MinIO Client mc commandline tool. See Test using MinIO Client mc for more information on using the mc commandline tool. For application developers, see https://min.io/docs/minio/linux/developers/minio-drivers.html to view MinIO SDKs for supported languages.\n\nNOTE: Standalone MinIO servers are best suited for early development and evaluation. Certain features such as versioning, object locking, and bucket replication require distributed deploying MinIO with Erasure Coding. For extended development and production, deploy MinIO with Erasure Coding enabled - specifically, with a minimum of 4 drives per MinIO server. See MinIO Erasure Code Overview for more complete documentation.\n\nMinIO strongly recommends against using compiled-from-source MinIO servers for production environments.\n\nDeployment Recommendations\n\nAllow port access for Firewalls\n\nBy default MinIO uses the port 9000 to listen for incoming connections. If your platform blocks the port by default, you may need to enable access to the port.\n\nufw\n\nFor hosts with ufw enabled (Debian based distros), you can use ufw command to allow traffic to specific ports. Use below command to allow access to port 9000\n\nsh\nufw allow 9000\n\nBelow command enables all incoming traffic to ports ranging from 9000 to 9010.\n\nsh\nufw allow 9000:9010/tcp\n\nfirewall-cmd\n\nFor hosts with firewall-cmd enabled (CentOS), you can use firewall-cmd command to allow traffic to specific ports. Use below commands to allow access to port 9000\n\nsh\nfirewall-cmd --get-active-zones\n\nThis command gets the active zone(s). Now, apply port rules to the relevant zones returned above. For example if the zone is public, use\n\nsh\nfirewall-cmd --zone=public --add-port=9000/tcp --permanent\n\nNote that permanent makes sure the rules are persistent across firewall start, restart or reload. Finally reload the firewall for changes to take effect.\n\nsh\nfirewall-cmd --reload\n\niptables\n\nFor hosts with iptables enabled (RHEL, CentOS, etc), you can use iptables command to enable all traffic coming to specific ports. Use below command to allow\naccess to port 9000\n\nsh\niptables -A INPUT -p tcp --dport 9000 -j ACCEPT\nservice iptables restart\n\nBelow command enables all incoming traffic to ports ranging from 9000 to 9010.\n\nsh\niptables -A INPUT -p tcp --dport 9000:9010 -j ACCEPT\nservice iptables restart\n\nTest MinIO Connectivity\n\nTest using MinIO Console\n\nMinIO Server comes with an embedded web based object browser. Point your web browser to http://127.0.0.1:9000 to ensure your server has started successfully.\n\nNOTE: MinIO runs console on random port by default, if you wish to choose a specific port use --console-address to pick a specific interface and port.\n\nThings to consider\n\nMinIO redirects browser access requests to the configured server port (i.e. 127.0.0.1:9000) to the configured Console port. MinIO uses the hostname or IP address specified in the request when building the redirect URL. The URL and port must be accessible by the client for the redirection to work.\n\nFor deployments behind a load balancer, proxy, or ingress rule where the MinIO host IP address or port is not public, use the MINIO_BROWSER_REDIRECT_URL environment variable to specify the external hostname for the redirect. The LB/Proxy must have rules for directing traffic to the Console port specifically.\n\nFor example, consider a MinIO deployment behind a proxy https://minio.example.net, https://console.minio.example.net with rules for forwarding traffic on port :9000 and :9001 to MinIO and the MinIO Console respectively on the internal network. Set MINIO_BROWSER_REDIRECT_URL to https://console.minio.example.net to ensure the browser receives a valid reachable URL.\n\nSimilarly, if your TLS certificates do not have the IP SAN for the MinIO server host, the MinIO Console may fail to validate the connection to the server. Use the MINIO_SERVER_URL environment variable  and specify the proxy-accessible hostname of the MinIO server to allow the Console to use the MinIO server API using the TLS certificate.\n\nFor example: export MINIO_SERVER_URL="https://minio.example.net"\n\n| Dashboard                                                                                   | Creating a bucket                                                                           |\n| -------------                                                                               | -------------                                                                               |\n|  |  |\n\nTest using MinIO Client mc\n\nmc provides a modern alternative to UNIX commands like ls, cat, cp, mirror, diff etc. It supports filesystems and Amazon S3 compatible cloud storage services. Follow the MinIO Client Quickstart Guide for further instructions.\n\nUpgrading MinIO\n\nUpgrades require zero downtime in MinIO, all upgrades are non-disruptive, all transactions on MinIO are atomic. So upgrading all the servers simultaneously is the recommended way to upgrade MinIO.\n\nNOTE: requires internet access to update directly from https://dl.min.io, optionally you can host any mirrors at https://my-artifactory.example.com/minio/\n\nFor deployments that installed the MinIO server binary by hand, use mc admin update\n\nsh\nmc admin update <minio alias, e.g., myminio>\n\nFor deployments without external internet access (e.g. airgapped environments), download the binary from https://dl.min.io and replace the existing MinIO binary let\'s say for example /opt/bin/minio, apply executable permissions chmod +x /opt/bin/minio and proceed to perform mc admin service restart alias/.\n\nFor installations using Systemd MinIO service, upgrade via RPM/DEB packages parallelly on all servers or replace the binary lets say /opt/bin/minio on all nodes, apply executable permissions chmod +x /opt/bin/minio and process to perform mc admin service restart alias/.\n\nUpgrade Checklist\n\nTest all upgrades in a lower environment (DEV, QA, UAT) before applying to production. Performing blind upgrades in production environments carries significant risk.\n\nRead the release notes for MinIO before performing any upgrade, there is no forced requirement to upgrade to latest release upon every release. Some release may not be relevant to your setup, avoid upgrading production environments unnecessarily.\n\nIf you plan to use mc admin update, MinIO process must have write access to the parent directory where the binary is present on the host system.\n\nmc admin update is not supported and should be avoided in kubernetes/container environments, please upgrade containers by upgrading relevant container images.\n\nWe do not recommend upgrading one MinIO server at a time, the product is designed to support parallel upgrades please follow our recommended guidelines.\n\nExplore Further\n\nMinIO Erasure Code Overview\n\nUse mc with MinIO Server\n\nUse minio-go SDK with MinIO Server\n\nThe MinIO documentation website\n\nContribute to MinIO Project\n\nPlease follow MinIO Contributor\'s Guide\n\nLicense\n\nMinIO source is licensed under the GNU AGPLv3.\n\nMinIO documentation is licensed under CC BY 4.0.\n\nLicense Compliance', metadata={'source': 's3://web-documentation/MinIO_Quickstart.md'}),
     Document(page_content='This is a sample document\n\n...\n\nSample Document\n\neof', metadata={'source': 's3://web-documentation/test-file-1.md'}),
     Document(page_content='This is a sample document\n\n...\n\nSample Document\n\neof', metadata={'source': 's3://web-documentation/test-file-2.md'})]



The above output lists the content of the specific bucket directory.

---

---

## #2 - How to use the Langchain S3 File Loader

Resources were accessing:
Endpoint: https://play.min.io
Bucket: "web-documentation"

#### Objective
Load a single document from an S3 file.

#### Steps
1. Import `S3FileLoader` from `langchain_community.document_loaders`.
2. Configure MinIO credentials and endpoint.
3. Initialize `S3FileLoader` with the specified bucket, file key, endpoint URL, AWS access keys, and SSL usage.
4. Load a single document from the specified file.

#### Code Block


```python
from langchain_community.document_loaders.s3_file import S3FileLoader

## MinIO Configuration
endpoint = 'play.min.io:9000'
access_key = 'minioadmin'
secret_key = 'minioadmin'
use_ssl = True

## Initializing the S3FileLoader
file_loader = S3FileLoader(
    bucket='web-documentation', 
    key='test-file-1.md', 
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key, 
    aws_secret_access_key=secret_key, 
    use_ssl=use_ssl
)

## Load a single document
document = file_loader.load()
document
```




    [Document(page_content='This is a sample document\n\n...\n\nSample Document\n\neof', metadata={'source': 's3://web-documentation/test-file-1.md'})]



The above output lists the content of the specified bucket diretory.

---

---

# #3 - Utilizing the S3 File Loader (with OpenAI API for Document Summary)


```python
from langchain_community.document_loaders.s3_file import S3FileLoader

from langchain_community.chat_models import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
import os

# Set your OpenAI API Key here
#os.environ['OPENAI_API_KEY'] = 'your-api-key'

# MinIO Configuration
endpoint = 'play.min.io:9000'
access_key = 'minioadmin'
secret_key = 'minioadmin'
use_ssl = True

# Initializing the S3FileLoader
file_loader = S3FileLoader(
    bucket='web-documentation', 
    key='MinIO_Quickstart.md', 
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key, 
    aws_secret_access_key=secret_key, 
    use_ssl=use_ssl
)

# Define LLM Setup
model = ChatOpenAI(temperature=0, model="gpt-4-1106-preview")

# Define the prompt template for summarization
template = "Summarize the following document: {context}"
prompt = ChatPromptTemplate.from_template(template)

# Load the document
loaded_documents = file_loader.load()

# Check if documents are loaded and extract the text content from the first document
if loaded_documents:
    document_text = loaded_documents[0].page_content

    # Define the Chain
    chain = (
        RunnableLambda(lambda x: {"context": document_text})
        | prompt
        | model
        | StrOutputParser()
    )

    # Execute Chain Synchronously
    summary = chain.invoke(None)
    print("Summary:", summary)
else:
    print("No documents loaded.")
```

    Summary: The MinIO Quickstart Guide provides instructions for setting up MinIO, a high-performance object storage system compatible with Amazon S3 APIs. It is suitable for machine learning, analytics, and application data workloads and is released under the GNU Affero General Public License v3.0.
    
    The guide covers installation methods for various platforms:
    
    - **Container Installation**: Instructions for running MinIO as a container using `podman` or `docker` commands.
    - **macOS**: Steps to install MinIO using Homebrew or binary download.
    - **GNU/Linux**: Instructions for running MinIO on different Linux architectures using `wget` to download the binary.
    - **Microsoft Windows**: Steps to download and run MinIO using the Windows executable.
    - **Install from Source**: For developers and advanced users, instructions to compile and run MinIO from source using Go.
    
    For all installations, MinIO starts with default credentials (minioadmin:minioadmin) and can be accessed via the MinIO Console at `http://127.0.0.1:9000`. Users can create buckets, upload objects, and browse data using the console or any S3-compatible tool like the MinIO Client (`mc`).
    
    The guide emphasizes that standalone MinIO servers are suitable for development and evaluation, but for production, MinIO should be deployed with Erasure Coding and a minimum of 4 drives per server.
    
    Additional sections include:
    
    - **Deployment Recommendations**: Firewall configuration to allow traffic to MinIO's default port (9000).
    - **Test MinIO Connectivity**: Using the MinIO Console and MinIO Client (`mc`) to test server functionality.
    - **Upgrading MinIO**: Guidelines for upgrading MinIO with zero downtime, including using `mc admin update` and handling upgrades in air-gapped environments.
    
    The guide also provides links to further documentation, SDKs for developers, and contribution guidelines for the MinIO project.
    
    Lastly, it mentions the licensing for MinIO source code (GNU AGPLv3) and documentation (CC BY 4.0).


The above output targets the specified file and sends the content to OpenAI API to return a summary.

---

---

# #4 - Utilizing the S3 Directory Loader (with OpenAI API for Document Summary)


```python
from langchain_community.document_loaders.s3_directory import S3DirectoryLoader

from langchain_community.chat_models import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
import os

# Set your OpenAI API Key here
#os.environ['OPENAI_API_KEY'] = 'your-api-key'

# MinIO Configuration
endpoint = 'play.min.io:9000'
access_key = 'minioadmin'
secret_key = 'minioadmin'
use_ssl = True


# Initializing the S3DirectoryLoader
directory_loader = S3DirectoryLoader(
    bucket='web-documentation', 
    prefix='',  # Adjust the prefix as needed
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key, 
    aws_secret_access_key=secret_key, 
    use_ssl=use_ssl
)

# Define LLM Setup
model = ChatOpenAI(temperature=0, model="gpt-4-1106-preview")

# Define the structured prompt template for summarization
structured_prompt_template = """
Summarize the following document '{document_name}':
{context}

Please provide the summary and key points.
"""
prompt = ChatPromptTemplate.from_template(structured_prompt_template)

# Load documents from the directory
loaded_documents = directory_loader.load()

# Initialize structured output
structured_output = ""

# Check if documents are loaded
if loaded_documents:
    for index, doc in enumerate(loaded_documents):
        document_name = f"Document {index + 1} - {doc.metadata.get('name', 'Unknown Document')}"
        document_text = doc.page_content

        # Define the Chain for each document
        chain = (
            RunnableLambda(lambda x: {"document_name": document_name, "context": document_text})
            | prompt
            | model
            | StrOutputParser()
        )

        # Execute Chain Synchronously for each document
        summary = chain.invoke(None)
        structured_output += f"\n{document_name}\n{summary}\n"

    print("Structured Summaries:", structured_output)
else:
    print("No documents loaded.")
```

    Structured Summaries: 
    Document 1 - Unknown Document
    Document 1 appears to be a comprehensive Quickstart Guide for MinIO, a high-performance object storage system that is compatible with the Amazon S3 API. It is released under the GNU Affero General Public License v3.0 and is suitable for machine learning, analytics, and application data workloads.
    
    The guide includes instructions for running MinIO on various platforms, including bare metal, container installations, Kubernetes, macOS, GNU/Linux, and Microsoft Windows. It emphasizes that standalone MinIO servers are ideal for development and evaluation, but for production environments, MinIO should be deployed with Erasure Coding and a minimum of 4 drives per server.
    
    For container installations, it provides commands to run MinIO using `podman` or `docker`. For macOS, it suggests using Homebrew for installation or downloading the binary directly. For GNU/Linux and Windows, it provides download links for the respective binaries and instructions on how to run them.
    
    The guide also includes a section on testing MinIO connectivity using the MinIO Console and the MinIO Client (`mc`) command-line tool. It provides URLs for accessing the MinIO SDKs for various programming languages.
    
    Additionally, the guide covers deployment recommendations, such as allowing port access through firewalls and testing MinIO connectivity. It provides detailed instructions for upgrading MinIO with minimal downtime, emphasizing the importance of testing upgrades in non-production environments first.
    
    Finally, the guide mentions that the MinIO source code is licensed under the GNU AGPLv3, and the documentation is licensed under CC BY 4.0. It also includes a contributor's guide for those interested in contributing to the MinIO project.
    
    Document 2 - Unknown Document
    The document provided is labeled as 'Document 2 - Unknown Document' and appears to be a placeholder or template rather than a document containing substantive content. The text includes the phrases "This is a sample document" and "Sample Document," followed by "eof," which typically stands for "end of file." There are no key points or detailed information to summarize, as the content is simply indicative of a generic or blank document.
    
    Document 3 - Unknown Document
    As an AI, I don't have access to external documents, including 'Document 3 - Unknown Document'. However, based on the text you've provided, it appears to be a placeholder or template rather than an actual document with content. The text "This is a sample document" followed by "Sample Document" and "eof" (end of file) suggests that there is no substantive content to summarize. If there is more specific content within the document that you would like to be summarized, please provide the text, and I'll be happy to help.
    


The above output summarizes documents with OpenAI API call using Prompt Templating. 


---

# GRAVEYARD


## Using Langchain S3 Loaders vs Using MinIO Client

### LangChain S3 Directory Loader

LangChain's S3 Directory Loader simplifies the process of loading documents from an S3-compatible service, including MinIO. It's designed for ease of integration within the LangChain framework.

```python
from langchain.document_loaders import S3DirectoryLoader

# Initialize the S3 Directory Loader with MinIO credentials
s3_loader = S3DirectoryLoader(
    bucket='your-bucket-name',
    prefix='your-directory-prefix',
    aws_access_key_id='minio-access-key',
    aws_secret_access_key='minio-secret-key',
    endpoint_url='minio-server-url',
    use_ssl=True  # Depending on your MinIO setup
)

# Load documents from the specified S3/MinIO directory
documents = s3_loader.load()
```

In this setup, you initialize `S3DirectoryLoader` with your MinIO bucket and credentials. The `load()` method then retrieves documents from the specified bucket and directory.

### MinIO Client

The MinIO client (`minio`) allows more direct and detailed interaction with MinIO. It offers broader functionality, including uploading, downloading, and managing objects and buckets.

```python
from minio import Minio

# Initialize the MinIO client
minio_client = Minio(
    'minio-server-url',
    access_key='minio-access-key',
    secret_key='minio-secret-key',
    secure=True  # Use False if not using SSL
)

# List objects in a bucket
objects = minio_client.list_objects('your-bucket', prefix='your-directory-prefix', recursive=True)

for obj in objects:
    # Process each object - e.g., download or read content
    print(obj.object_name)
```

With the MinIO client, you have more control over the operations you perform with your MinIO server. The `list_objects` method here lists all objects in the specified bucket and prefix, and you can then download or manage these objects as needed.

### Key Differences

- **Integration**: LangChain's loader is specifically designed for easy integration within LangChain chains and agents, whereas the MinIO client offers more general-purpose object storage interactions.
- **Functionality**: The MinIO client provides a wider range of functionalities, like bucket management, object upload/download, etc., while LangChain's loader focuses on loading documents for processing within LangChain.
- **Use Case**: Use LangChain's loader for streamlined document loading within LangChain applications. Use the MinIO client for more varied and detailed interactions with your MinIO storage.
