# RETRANS

**Reliable Event Transport, Routing, and Normalization System**  
*An Iron Signal Systems project*

**Collect from many sources. Deliver only what each receiver needs.**

## Overview

RETRANS is a log aggregation, processing, and forwarding service written in Go, designed initially for Microsoft Windows Server.

Its purpose is straightforward: collect events from different systems, normalize and filter them according to configurable policies, and securely deliver the selected records to one or more receiving platforms.

RETRANS is intended to work between existing log sources and the systems that consume their data. It does not require every receiver to accept the same events, and it does not require organizations to replace their existing monitoring or analysis tools.

## Why RETRANS?

Log collection is often more complicated than it needs to be. Organizations may have Windows events, network equipment, infrastructure services, and application logs that must reach different destinations. Each destination may support different event types or require a different format.

Sending everything everywhere creates unnecessary traffic, processing, and storage demands. Maintaining separate forwarding solutions for every destination adds operational complexity and makes delivery failures harder to identify.

RETRANS is being built to provide a focused alternative: **one service for collection and processing, with independent policies for each receiver.**

## What RETRANS Is Designed to Do

- **Aggregate multiple sources.** Bring together Windows events, including forwarded events, syslog, and other supported application and infrastructure logs.
- **Normalize event data.** Present common information consistently across different source formats while preserving the original records where required.
- **Filter selectively.** Allow only the events a receiving system is configured to accept, without forcing the same rules on every destination.
- **Route independently.** Send different sets of events to different receivers, each with its own delivery policy.
- **Protect data in transit.** Use encrypted outbound connections, validate receiver identity, and support externally provisioned certificates, including client certificates when required.
- **Handle interruptions predictably.** Buffer selected events, retry delivery, and make backlogs or failures visible rather than allowing them to go unnoticed.
- **Operate efficiently.** Favor controlled resource consumption and minimal supporting infrastructure.

### Different Receivers, Different Requirements

A receiving platform may accept only a defined set of authentication or security events. Another may need those same events alongside DNS activity, DHCP activity, or application logs.

RETRANS is designed to support both from the same collection service. **Each receiver gets the records selected by its own policy**, and changing one receiver's policy does not require changing the others.

## Design Principles

**Simple by purpose.** RETRANS concentrates on collecting, processing, and transporting log data rather than becoming a full monitoring platform.

**Receiver-independent.** No single SIEM, vendor, or destination format defines how the core service operates.

**Configurable.** Event selection and delivery behavior should be adjustable without modifying or recompiling the application.

**Secure by design.** Encrypted delivery and externally managed trust material are foundational requirements, not optional add-ons.

**Predictable under load.** Processing, memory use, and buffering should remain bounded so that RETRANS does not compromise the stability of its host.

**Failures must be visible.** Administrators should be able to distinguish intentionally filtered records from processing failures and delivery interruptions.

**Independent deployment.** RETRANS is intended to run on infrastructure controlled by its operator, without a required hosted service or external database.

## What RETRANS Is Not

RETRANS is **not a SIEM, threat-detection engine, investigation platform, or long-term log archive**. Those responsibilities belong to the systems that receive and analyze the data.

Its responsibility is to ensure that the right records are prepared and delivered to the right destinations, with clear operational visibility into that process.

## Platform and Project Status

Microsoft Windows Server is the initial target platform. The core design is intended to remain portable so that support for additional operating systems can be considered later.

**Status: Early design.** This README describes the project's intended direction and capabilities. RETRANS is not yet a production-ready release.

Detailed architecture, configuration, transport contracts, security decisions, and acceptance testing will be maintained separately from this overview in the repository's technical documentation.

## License

Copyright 2026 John Joseph Wood.

RETRANS is licensed under the **Apache License 2.0**. See the [LICENSE](LICENSE) file for the full license terms.
