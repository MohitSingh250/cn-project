# Scripts — Setup & Verification Guides

This directory contains step-by-step documentation for every aspect of the project setup, testing, and failure demonstration.

Each guide contains the exact commands that were used, the expected output, and explanations of what each step accomplishes at the network layer.

---

## Directory Index

### `setup/` — Machine Setup
| File | Description |
|------|-------------|
| [`mac1-setup.md`](setup/mac1-setup.md) | Complete setup guide for Mac 1 (Abhishek) |
| [`mac2-setup.md`](setup/mac2-setup.md) | Complete setup guide for Mac 2 (Mohit) |

### `backend/` — Backend Creation
| File | Description |
|------|-------------|
| [`create-backend-a.md`](backend/create-backend-a.md) | How Backend A was created |
| [`create-backend-b.md`](backend/create-backend-b.md) | How Backend B was created |
| [`start-backends.md`](backend/start-backends.md) | Starting both backends |

### `dns/` — DNS Configuration
| File | Description |
|------|-------------|
| [`dnsmasq-installation.md`](dns/dnsmasq-installation.md) | Installing dnsmasq on Mac 1 |
| [`dnsmasq-config.md`](dns/dnsmasq-config.md) | Configuring private DNS records |
| [`dns-verification.md`](dns/dns-verification.md) | Verifying DNS resolution |

### `tls/` — TLS Certificate Setup
| File | Description |
|------|-------------|
| [`local-ca.md`](tls/local-ca.md) | Creating the local Certificate Authority |
| [`certificate-generation.md`](tls/certificate-generation.md) | Generating server certificates with SAN |
| [`certificate-verification.md`](tls/certificate-verification.md) | Verifying the certificate chain |

### `nginx/` — nginx Configuration
| File | Description |
|------|-------------|
| [`nginx-installation.md`](nginx/nginx-installation.md) | Installing nginx on Mac 2 |
| [`nginx-config.md`](nginx/nginx-config.md) | Configuring reverse proxy & load balancer |
| [`nginx-verification.md`](nginx/nginx-verification.md) | Verifying nginx is running |

### `testing/` — Test Procedures
| File | Description |
|------|-------------|
| [`dns-tests.md`](testing/dns-tests.md) | DNS resolution tests |
| [`https-tests.md`](testing/https-tests.md) | HTTPS / TLS verification tests |
| [`load-balancing-tests.md`](testing/load-balancing-tests.md) | Load balancing verification |
| [`caching-tests.md`](testing/caching-tests.md) | HTTP caching tests |

### `failure-demo/` — Failure Demonstrations
| File | Description |
|------|-------------|
| [`backend-a-failure.md`](failure-demo/backend-a-failure.md) | Simulating Backend A going down |
| [`recovery.md`](failure-demo/recovery.md) | Recovering from failure |
