---
topic: "Software Design and Architecture, AWS Cloud Architecture, GCP Cloud Architecture"
perspective: "Windows Systems Engineer, Beginner"
date: 2026-04-03
staleness: 8
plugins:
  - context7
  - microsoft-docs
confidence: high
source_count: 82
tags:
  - ketchup
  - software-architecture
  - aws
  - gcp
  - cloud-architecture
  - microservices
  - iac
  - windows-to-cloud
  - migration
---

# Software Design and Cloud Architecture: A Guide for Windows Systems Engineers

> [!abstract] TL;DR
> Cloud architecture is not a foreign language — it maps directly onto what you already know from managing Windows Server, Active Directory, and on-prem infrastructure. Microservices are your monolithic IIS app broken into independent pieces. VPCs are your VLANs. IAM is your Active Directory. Infrastructure as Code is your Server Manager wizards turned into text files. AWS and GCP offer the same core concepts with different organizational models, and your PowerShell skills transfer to both.

## Why This Matters to You

You manage Windows systems. You know IIS, SQL Server, Active Directory, DNS, DHCP, Group Policy, and PowerShell. In 2018, your world was primarily on-premises — physical servers or VMware VMs, capital expenditure budgets, and maybe a DR site across town.

Eight years later, the industry has moved. Over 95% of new digital workloads now run on cloud-native platforms, up from 30% in 2021 ([Webelight](https://www.webelight.com/blog/choosing-the-right-software-architecture-pattern-in-2025)). Kubernetes — a technology that barely existed in Windows shops in 2018 — is now deployed in production by 80% of IT organizations ([Orkes](https://orkes.io/blog/software-architecture-evolution/)). The cloud certifications that matter most (AZ-104, AWS SAA) are now standard hiring requirements for infrastructure roles.

The good news: moving to cloud doesn't mean throwing away everything you know. It means applying what you know in a new environment. This report bridges every concept from where you stand.

## What This Is NOT

> [!warning] Wrong mental models to discard

- **"Cloud" is not "someone else's computer."** It's a fundamentally different operational model — on-demand, API-driven, disposable infrastructure. The hardware is someone else's; the architecture is yours.
- **Microservices are not just "smaller programs."** They're independently deployable services with their own databases, APIs, and lifecycles. The independence is the point.
- **"Serverless" does not mean "no servers."** It means you don't manage them. Servers exist; you just never see or patch them.
- **AWS and GCP are not interchangeable.** Same concepts, different implementations. IAM works differently, networking is structured differently, billing is organized differently. Learning one does not mean you know the other.
- **You don't have to learn everything at once.** Start with VMs in one cloud (they're just VMs), then expand.

## Core Concepts: Software Design Fundamentals

### From One Big App to Many Small Ones

Think about a typical IIS deployment you managed in 2018: your web application lives in one Application Pool, talks to one SQL Server database, and handles everything — authentication, business logic, reporting, file uploads — as a single unit. That is a **monolith**. Any change means redeploying the whole thing. If the reporting module eats all the CPU, it takes down your login page too.

**Microservices** break that application apart. Instead of one IIS site doing everything, you have separate, small services — one handles authentication, one handles orders, one handles reporting — each running independently, each with its own database if needed. If reporting falls over, orders keep processing. Each service can be updated without touching the others. This pattern emerged around 2014 ([AWS](https://aws.amazon.com/compare/the-difference-between-soa-microservices/)) and 77% of organizations adopting it report improved scalability ([Webelight](https://www.webelight.com/blog/choosing-the-right-software-architecture-pattern-in-2025)).

> [!info] The progression you missed
> Monoliths → **SOA** (late 1990s, heavy middleware, Enterprise Service Bus) → **Microservices** (2014+, lightweight, independent) → **Serverless/Event-driven** (functions triggered by events, no server management). You left during the monolith era. The industry is now firmly in microservices + serverless ([Orkes](https://orkes.io/blog/software-architecture-evolution/)).

### Containers: VMs, But Lighter

You know VMware. A VM gives you an isolated OS on shared hardware. A **container** is a similar idea stripped down: it packages just your application and its dependencies, shares the host OS kernel, and starts in seconds. Where a VM might be 20 GB, a container image might be 200 MB _(unsourced)_.

**Kubernetes** (K8s) is the orchestration layer — scheduling containers across servers, restarting failed ones, scaling up under load. Think of it as Task Scheduler + Failover Clustering + load balancing, designed for containers from the ground up.

> [!tip] This didn't exist in your 2018 world
> Docker existed in 2018 but was niche in Windows shops. Kubernetes was experimental. Both are now standard infrastructure.

### APIs: How Services Talk

"API" means a defined way for one piece of software to talk to another. Your IIS apps almost certainly consumed APIs — a payment gateway, a mapping service. Three styles matter now:

| Style | Best for | Analogy |
|-------|----------|---------|
| **REST** | Public APIs, web services | Standard HTTP verbs (GET/POST/PUT/DELETE) against URLs. Simple, universal. |
| **GraphQL** | Flexible data queries | Single endpoint, client specifies exactly what data it needs. |
| **gRPC** | Fast internal service-to-service | Binary format over HTTP/2. Much faster than REST, but not human-readable. |

Modern strategy: REST or GraphQL for public-facing APIs, gRPC for internal communication between your own microservices ([Design Gurus](https://www.designgurus.io/blog/rest-graphql-grpc-system-design)).

### Infrastructure as Code: Server Manager Becomes a Text File

Installing a Windows role in 2018 meant opening Server Manager, clicking through wizards, and documenting what you did in a Word file. **Infrastructure as Code** (IaC) replaces those clicks with a text file describing your desired infrastructure state.

```hcl
# Terraform example: create a VM (declarative — you describe WHAT, not HOW)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = { Name = "WebServer" }
}
```

Two approaches: **declarative** (describe what you want, the tool figures out how — like a GPO defining desired state) and **imperative** (write the steps — like a PowerShell script). Most modern IaC is declarative. Key tools: **Terraform** (cross-cloud), **CloudFormation** (AWS), **Bicep** (Azure) ([AWS](https://aws.amazon.com/what-is/iac/); [Red Hat](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac)).

If you touched PowerShell DSC, you already understand the concept.

---

## Cloud Architecture: The Translation

### The Three Service Models

| Model | What the provider manages | What you manage | Your analogy |
|-------|--------------------------|-----------------|--------------|
| **IaaS** | Hardware, hypervisor, networking | OS, apps, data, patches | Renting a server room instead of building one. Your VMware VMs, but hosted. |
| **PaaS** | All of IaaS + OS + runtime | Your application code + data | Someone else manages IIS. You just deploy your app. |
| **SaaS** | Everything | Your data + who has access | Office 365. You already use this. |

([AWS](https://aws.amazon.com/types-of-cloud-computing/); [Google Cloud](https://cloud.google.com/learn/paas-vs-iaas-vs-saas); [Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/describe-cloud-service-types/))

### Regions, Availability Zones, and Edge Locations

Your primary datacenter + DR site is roughly one cloud region with two Availability Zones — except cloud gives you at least three, with failover automation built in.

- **AWS**: 39 regions, each with 3+ AZs (physically separated within ~100km). Hundreds of CloudFront edge locations ([AWS](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)).
- **GCP**: 47 regions, 127 zones, 202 edge locations ([Economize](https://www.economize.cloud/resources/gcp/regions-zones-map/)).

### Shared Responsibility Model

The hosting company secures the building, power, and hardware. You secure your servers, data, and identities. This shifts by service model: on IaaS, you patch the OS (same as today). On PaaS, the provider does. On SaaS, almost everything is theirs except your data and access control ([AWS](https://aws.amazon.com/compliance/shared-responsibility-model/); [Microsoft](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)).

### Resource Hierarchy: It's Just OUs

| Cloud | Structure | AD Analogy |
|-------|-----------|------------|
| **AWS** | Organization → OUs → Accounts → Resources | Forest → OUs → Computers |
| **GCP** | Organization → Folders → Projects → Resources | Forest → OUs → Computers |
| **Azure** | Tenant → Management Groups → Subscriptions → Resource Groups → Resources | Forest → OUs → Sub-OUs → Computers |

([AWS](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html); [GCP](https://docs.cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy))

### Billing: From CapEx to the Meter

| Model | What it is | Analogy |
|-------|-----------|---------|
| **Pay-as-you-go** | Per hour/second, no commitment | Renting a hotel room nightly |
| **Reserved** | 1-3 year commitment, 20-75% discount | Signing a lease |
| **Spot/Preemptible** | Up to 90% savings, provider can reclaim anytime | Standby airline tickets |

([Spot.io](https://spot.io/resources/cloud-cost/leveraging-cloud-computing-pricing-models-for-greater-cost-efficiency/); [Absolute App Labs](https://absoluteapplabs.com/blog/pay-as-you-go-vs-reserved-instances-vs-spot-instances-explained/))

### Pets vs. Cattle

Your domain controller named DC01 — nursed through hardware migrations, service packs, and two near-disasters over five years — is a **pet**. Cloud VMs are **cattle**: numbered, disposable, replaced from a template when they fail. Nobody names them. Nobody logs in to make one-off changes. Configuration lives in code. **Immutable infrastructure** extends this: servers are never patched, only replaced from standard templates ([Kubermatic](https://www.kubermatic.com/blog/cloud-native-best-practices-2-why-cattle-not-pets/)).

This is one of the sharpest breaks from the on-prem mindset. Sit with it until it feels natural.

---

## AWS Cloud Architecture

### The Console

The **AWS Management Console** (console.aws.amazon.com) is a web UI — Server Manager crossed with Hyper-V Manager, but for cloud infrastructure. Every service has its own panel. Start here before automation.

### Core Services Mapped

| AWS Service | What it is | Your Windows Equivalent |
|-------------|-----------|------------------------|
| **EC2** | Virtual machines | Hyper-V VM, but in a browser |
| **Lambda** | Event-triggered code, no server | PowerShell script that runs on a trigger, server-free |
| **S3** | Unlimited object storage via URLs | `\\fileserver\share\` but accessed via `https://bucket.s3.amazonaws.com/` |
| **EBS** | Block storage for VMs | Virtual hard disk attached to a VM |
| **VPC** | Virtual network | VLAN in your datacenter |
| **Security Groups** | Stateful firewall rules on VMs | Windows Firewall rules, but allow-only (no explicit deny) |
| **Route 53** | Managed DNS | Your DNS server, globally distributed |
| **CloudFront** | CDN | Content caching at edge locations |
| **IAM** | Identity and access management | Active Directory, but no OU hierarchy |

([AWS Compute](https://aws.amazon.com/products/compute/); [AWS Storage](https://aws.amazon.com/products/storage/); [AWS VPC](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html))

### AWS IAM: AD But Different

No OU hierarchy. Permissions are **JSON policy documents** attached to users, groups, or roles. **IAM Roles** issue temporary credentials rather than static passwords — a time-limited token instead of a service account password ([AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)). Every resource has an **ARN** (Amazon Resource Name) — the equivalent of a Distinguished Name in AD ([AWS Identifiers](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html)).

### AWS Well-Architected Framework

Six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability ([AWS](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)). Think of it as Microsoft's best-practice documentation for Windows Server, but for cloud architecture.

### Free Tier

Practice without spending money. **Always Free**: 1M Lambda requests/month, 25GB DynamoDB. **12-month**: 750h EC2 t2/t3.micro, 5GB S3 ([AWS Free Tier](https://aws.amazon.com/free/)).

---

## GCP Cloud Architecture

### Where GCP Differs from AWS

GCP's compute lineup (Compute Engine, App Engine, Cloud Functions, GKE) parallels AWS closely. The meaningful differences are:

**1. IAM Model — Think NTFS, Not AD Users**

In AWS, permissions travel with the identity (policies attached to users/roles). In GCP, **you grant a role on a specific resource** — the permission lives on the resource. This is closer to NTFS: you set permissions on the folder, not the user.

Model: **Principal + Role + Resource**. Roles come in three tiers: Basic (Owner/Editor/Viewer), Predefined (service-specific), and Custom ([GCP IAM](https://cloud.google.com/iam/docs/overview); [StrongDM](https://www.strongdm.com/blog/gcp-iam-roles)).

**Service accounts** work like AD service accounts — identities for applications, not people ([Context7](https://docs.cloud.google.com/iam/docs/reference/rpc/google.iam.admin.v1)).

**2. Subnets Are Global Within a Region**

In AWS, subnets are per-Availability Zone. In GCP, a subnet spans the **entire region** — all zones automatically. Imagine if your VLAN replicated itself across all three of your datacenters without configuration.

**3. Project-Centric Billing**

Every resource lives inside a **Project**, and each Project links to a billing account. Costs aggregate at the Folder level. More explicit than AWS's account-per-environment approach ([GCP Billing](https://docs.cloud.google.com/billing/docs/concepts)).

### GCP Free Tier

**f1-micro VM in US regions is always free** — never expires, no trial period. Also: 5GB Cloud Storage, 1M Cloud Functions invocations, 1TB BigQuery queries monthly ([GCP Free Tier](https://cloud.google.com/free); [GCP Free Features](https://docs.cloud.google.com/free/docs/free-cloud-features)). This is your permanent practice lab.

---

## Migration: How a Windows Shop Moves to Cloud

### The 6 Rs

| Strategy | What it means | When to use |
|----------|--------------|-------------|
| **Rehost** (lift-and-shift) | Move VMs as-is | Quick win, learn the platform |
| **Replatform** | Shift to managed services (SQL Server → Azure SQL) | Reduce ops without rewriting |
| **Refactor** | Redesign for cloud-native | Long-term optimization |
| **Repurchase** | Switch to SaaS | Replace custom apps with products |
| **Retain** | Keep on-prem | Not everything needs to move |
| **Retire** | Decommission | Kill what's not needed |

([C# Corner](https://www.c-sharpcorner.com/article/6-rs-strategies/))

Start with Rehost. Get your workloads running in the cloud. Optimize later.

### Your Skills Transfer

- **PowerShell** works natively on all three clouds: Azure PowerShell, AWS Tools for PowerShell, Google Cloud PowerShell ([HashiCorp](https://www.hashicorp.com/en/resources/managing-terraform-cloud-with-powershell))
- **Active Directory** syncs to the cloud via **Entra Cloud Sync** (formerly Azure AD Connect) — same credentials, same groups ([Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/tutorial-pilot-aadc-aadccp))
- **Terraform** provisions cross-cloud infrastructure from text files, with PowerShell-friendly workflows ([Terraform](https://developer.hashicorp.com/terraform/cloud-docs/migrate))
- **Azure Arc** manages on-prem servers from the Azure portal — extend the cloud control plane to your server room without moving anything ([Azure Arc](https://azure.microsoft.com/en-us/products/azure-arc))

### Certifications

| Cert | Focus | Time | Best for |
|------|-------|------|----------|
| **AZ-104** | Azure administration (VMs, networking, storage, identity) | 6-10 weeks | Windows admins — closest to what you already do |
| **AWS SAA-C03** | Solutions architecture (design, trade-offs) | 6-10 weeks | Second cert — architectural thinking |

([Azure AZ-104](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/); [CertEmpire](https://certempire.com/saa-c03-vs-az-104-cloud-certification/))

---

## Quick Reference

### Cloud Service Rosetta Stone

| Concept | AWS | GCP | Azure | Windows On-Prem |
|---------|-----|-----|-------|-----------------|
| VM | EC2 | Compute Engine | Virtual Machines | Hyper-V VM |
| Serverless functions | Lambda | Cloud Functions | Functions | (no equivalent) |
| Object storage | S3 | Cloud Storage | Blob Storage | File share (UNC) |
| Block storage | EBS | Persistent Disk | Managed Disk | Virtual hard disk |
| Virtual network | VPC | VPC | VNet | VLAN |
| DNS | Route 53 | Cloud DNS | DNS Zone | DNS Server role |
| Firewall rules | Security Groups | VPC Firewall Rules | NSG | Windows Firewall |
| Identity | IAM | IAM | Entra ID | Active Directory |
| IaC | CloudFormation | Deployment Manager | Bicep/ARM | PowerShell DSC |
| CLI | `aws` | `gcloud` | `az` | PowerShell |
| Container orchestration | EKS | GKE | AKS | (no equivalent) |
| CDN | CloudFront | Cloud CDN | Front Door | (no equivalent) |

### Design Patterns You'll Encounter

| Pattern | What it does | Analogy |
|---------|-------------|---------|
| **Strangler Fig** | Gradually replace monolith pieces with microservices | Migrating AD users one OU at a time |
| **Circuit Breaker** | Stop calling a failing service, fail fast | Auto-disabling a broken print queue |
| **Retry** | Automatically retry failed requests | Reconnecting a mapped drive |
| **CQRS** | Separate read and write operations | Different SQL views for reporting vs transactions |
| **Saga** | Coordinate multi-service transactions | Linked GPO changes that roll back together |

([Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/patterns/))

---

## Gotchas & Pitfalls

> [!danger] Things that WILL trip you up

1. **Don't lift-and-shift and stop there.** Rehosting is phase 1, not the destination. Leaving un-optimized VMs running 24/7 in the cloud costs more than on-prem _(~inferred: aggregate pattern from migration cost analyses)_.

2. **Cloud billing is relentless.** A VM you forgot to turn off bills by the hour _(unsourced)_. Set up billing alerts on day one. There is no "it's already paid for" in cloud — the meter is always running.

3. **IAM is not optional.** The default root account is like logging in as Domain Admin for everything _(~inferred: pattern-match from AWS IAM best practices documentation)_. Create individual IAM users/roles immediately. AWS and GCP both provide free identity management — use it.

4. **Security Groups / Firewall Rules default to deny.** Cloud security groups deny all inbound traffic by default until you explicitly allow it ([AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)). Your VM will seem "broken" until you open the right ports.

5. **Availability Zones are not automatic.** Deploying a VM in one AZ gives you zero redundancy _(unsourced)_. You must explicitly distribute across AZs for high availability. Cloud gives you the tools; it doesn't use them for you.

6. **GCP and AWS IAM are structurally different.** Don't assume what you learned in one applies to the other. AWS attaches policies to identities ([AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)); GCP grants roles on resources ([GCP IAM](https://cloud.google.com/iam/docs/overview)).

7. **"Serverless" still has cold starts.** Lambda/Cloud Functions may take several hundred milliseconds to spin up after idle periods _(unsourced)_. This matters for latency-sensitive applications.

8. **Terraform state files are critical.** The `.tfstate` file tracks what Terraform has created. Lose it or corrupt it and Terraform can't manage your infrastructure ([Terraform State](https://developer.hashicorp.com/terraform/cloud-docs/migrate)). Store it remotely (S3/GCS) from day one, not on your laptop.

9. **Your PowerShell ISE muscle memory works, but the cmdlets are different.** `Get-AzVM` (Azure), `Get-EC2Instance` (AWS) — same pattern, different nouns _(unsourced)_. Budget time for learning the specific module.

10. **Start with the free tier, but watch the clock.** AWS 12-month free tier items expire ([AWS Free Tier](https://aws.amazon.com/free/)). GCP always-free items don't ([GCP Free Tier](https://cloud.google.com/free)). Know which is which before you get a surprise bill.

---

## Further Reading

**Getting Started:**
- [AWS Free Tier](https://aws.amazon.com/free/) — Practice without spending
- [GCP Free Tier](https://cloud.google.com/free) — Always-free resources including a permanent VM
- [Azure for AWS Professionals](https://learn.microsoft.com/azure/architecture/aws-professional/) — Microsoft's official comparison guide
- [GCP for Azure/AWS Professionals](https://learn.microsoft.com/azure/architecture/gcp-professional/) — Google Cloud mapped to what you know

**Architecture Fundamentals:**
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html) — 6 pillars of good cloud design
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/guide/) — Patterns, reference architectures, best practices
- [Cloud Design Patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/) — Retry, Circuit Breaker, CQRS, and more
- [.NET Microservices Architecture eBook](https://dotnet.microsoft.com/download/thank-you/microservices-architecture-ebook) — Free, comprehensive (Microsoft)

**Migration:**
- [Azure Migrate](https://learn.microsoft.com/en-us/azure/migrate/concepts-migration-webapps) — Assessment and migration tools
- [AWS Application Discovery](https://docs.aws.amazon.com/application-discovery/latest/userguide/what-is-appdiscovery.html) — Inventory your on-prem environment
- [Terraform Getting Started](https://developer.hashicorp.com/terraform/cloud-docs/migrate) — Cross-cloud IaC

**Certifications:**
- [AZ-104: Azure Administrator](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/) — Your natural first cert
- [AWS SAA-C03: Solutions Architect](https://certempire.com/saa-c03-vs-az-104-cloud-certification/) — Architectural thinking
