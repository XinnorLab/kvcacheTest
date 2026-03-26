# Requirements for Software to Select NVMe Devices and Export Them over NVMe over Fabrics

## 1. Purpose of the Document

This document defines the requirements for software that must:

1. discover local NVMe devices on a host;
2. select NVMe devices based on defined criteria;
3. generate a JSON description of the selected devices;
4. export the selected devices over NVMe over Fabrics;
5. preserve the key identification attributes of those devices on the remote initiator;
6. support ANA states for multipath environments where multiple interfaces may exist between storage and client.

---

## 2. Goal of the Solution

The software must provide automatic discovery and filtering of local NVMe devices based on defined criteria, generate a JSON description of the selected devices, and publish those devices over NVMe over Fabrics in such a way that the remote initiator can access the same devices while preserving their key identification and operational attributes, including serial number, model, vendor, and other properties visible on the original local host.

In addition, the solution must support **Asymmetric Namespace Access (ANA)** state handling so that, in deployments with multiple storage-to-client interfaces, the initiator can correctly distinguish optimized and non-optimized paths and behave consistently in multipath scenarios.

---

## 3. General Problem Statement

The software must solve three related problems:

### 3.1. Discovery and Selection of NVMe Devices

The system must discover local NVMe devices and allow selection of devices according to defined filters.

### 3.2. Export of Selected Devices over NVMe over Fabrics

After selection, the system must export the devices over NVMe-oF so that the remote initiator can see the devices with their key attributes preserved as closely as possible to how they appear on the local host.

### 3.3. Representation of Multipath and ANA State Information

Where multiple interfaces or paths exist between the storage target and the client, the system must support ANA-aware publication and verification so that the initiator can correctly interpret path roles and availability.

---

## 4. Functional Requirements

## 4.1. NVMe Device Inventory

The system must discover local NVMe devices and collect the attributes required for filtering, publishing, and later verification.

At minimum, the following attributes must be available:

- PCIe address of the device;
- NUMA node association;
- model;
- vendor / manufacturer;
- serial number;
- firmware version;
- size / capacity;
- namespace information;
- controller information.

---

## 4.2. NVMe Device Filtering

The system must support selecting devices by one or more criteria at the same time.

Supported filtering criteria must include at least:

- membership in a specific NUMA node;
- exact device size;
- size range;
- vendor;
- model;
- PCIe locality;
- allow-list of devices;
- deny-list of devices;
- user-defined selection rules.

Filtering must produce a deterministic result.

---

## 4.3. JSON Description Generation

After applying the filters, the system must generate a JSON document that describes the selected device set.

This JSON must serve as the contract between the discovery/filtering stage and the export/publication stage.

The JSON must include at least:

- the list of selected NVMe devices;
- unique identifiers of those devices;
- key identification attributes;
- attributes required for export;
- data required for later validation on the initiator side.

---

## 4.4. Export over NVMe over Fabrics

The system must be able to export the selected devices over NVMe over Fabrics.

After export, a remote initiator must be able to connect to the exported devices and observe their key attributes preserved as closely as possible within the chosen implementation approach.

At minimum, the following characteristics must be preserved or correctly represented:

- serial number;
- model;
- vendor / manufacturer information;
- namespace identity;
- size;
- other significant identification fields visible on the local host.

---

## 4.5. Support for ANA States

The system must support **Asymmetric Namespace Access (ANA)** state handling for exported devices in deployments where multiple interfaces or access paths exist between storage and client.

The solution must satisfy the following ANA-related requirements:

- the export layer must be capable of exposing ANA-relevant path behavior where the target architecture supports it;
- the remote initiator must be able to observe path states in a way that is consistent with multipath operation;
- the solution must support correct distinction between preferred / optimized and non-optimized access paths where applicable;
- the verification flow must confirm that ANA behavior is presented consistently across multiple interfaces.

At minimum, the requirements must explicitly account for these logical ANA state classes:

- optimized path;
- non-optimized path;
- inaccessible path;
- state transition / change handling.

If the chosen implementation cannot expose every NVMe ANA state natively, the system must document which states are supported, how they are represented, and what behavior the initiator is expected to observe.

---

## 4.6. Remote Attribute Verification

The system must provide a way to verify that:

- the devices are reachable on the remote initiator;
- the devices correspond to the selected set;
- the key attributes on the initiator match the original local-host attributes, or are reproduced as closely as possible within the architectural constraints;
- in multipath configurations, the initiator sees the expected path topology and ANA-related behavior.

---

## 5. Expected Workflow

The target workflow must be as follows:

1. The system scans the local NVMe devices.
2. A user, CLI, API, or external component provides a filter.
3. The system determines the set of devices matching the filter.
4. The system generates a JSON description of the selected devices.
5. Based on that JSON, the system exports the devices over NVMe-oF.
6. The remote initiator connects to the exported devices.
7. The system or an external validation component verifies attribute consistency between the local host and the initiator.
8. In multipath environments, the system or validation component verifies ANA-related behavior and path-state visibility.

---

## 6. Input Data

The input data for the system includes:

- the local NVMe device configuration on the host;
- filtering parameters;
- NVMe-oF publication configuration;
- remote initiator parameters;
- rules for attribute consistency validation;
- where applicable, multipath and ANA-related publication rules.

---

## 7. Output Data

The output of the system includes:

- a JSON document describing the selected NVMe devices;
- publication configuration or publication result for NVMe-oF export;
- a list of exported devices;
- remote initiator discovery and connectivity results;
- attribute consistency validation results between local and remote representation;
- where applicable, multipath and ANA verification results.

---

## 8. Non-Functional Requirements

The system must satisfy the following non-functional requirements:

- the discovery and selection process must be reproducible;
- filtering must be deterministic;
- the JSON format must be stable and suitable for automated processing;
- the system must support extension of the filtering model;
- the system must support extension of the exported attribute set;
- the system must provide diagnosable error handling;
- the system must be suitable for automated testing;
- the solution must be usable as part of a larger automated workflow;
- multipath and ANA behavior must be testable and reproducible where the environment supports it.

---

## 9. Acceptance Criteria

The solution may be considered correct if all of the following are true:

1. The system discovers local NVMe devices and collects the required attributes.
2. The system correctly selects devices according to the specified filter.
3. The selection result is generated in JSON format.
4. The JSON contains sufficient data for publication and validation.
5. The selected devices are exported over NVMe-oF.
6. The remote initiator can see the exported devices.
7. The key device attributes on the initiator match the source attributes or are preserved as closely as possible within the selected architecture.
8. In multipath configurations, the initiator observes consistent ANA-related behavior.
9. The scenario can be reproduced with the same result.

---

## 10. Edge Cases and Special Scenarios

The system must correctly handle the following cases:

- the filter returns no devices;
- the filter returns multiple devices with overlapping partial attributes;
- some devices become unavailable at publication time;
- some attributes cannot be fully represented on the remote side;
- a device changes state between discovery and export;
- the system contains NVMe devices from different vendors and with different namespace layouts;
- some devices must be explicitly excluded from publication;
- the target is exposed over multiple interfaces and the client sees multiple access paths;
- one or more paths are intentionally presented as non-optimized;
- one or more paths become temporarily inaccessible;
- ANA state changes occur while the remote initiator remains connected.

---

## 11. Primary Value of the Solution

The primary value of the solution is not only to select NVMe devices and export them over NVMe over Fabrics, but also to preserve device identity when those devices are made remotely accessible.

In multipath deployments, the value also includes preserving meaningful access semantics, so the remote initiator can distinguish path roles and availability rather than seeing an opaque storage object without path-awareness.

The system must allow the remote side to perceive the exported resource as an NVMe device with its original identity preserved as closely as possible.

---

## 12. Expected Implementation Artifacts

The solution may include the following components:

- NVMe device discovery module;
- filtering module;
- JSON schema for selection results;
- NVMe-oF export / publication module;
- remote-side validation module;
- ANA and multipath validation module;
- CLI or API for running the full workflow;
- automated tests for filtering correctness;
- automated tests for publication correctness;
- automated tests for attribute preservation;
- automated tests for ANA-aware multipath behavior.

---

## 13. Short Problem Statement

The goal is to develop software that discovers local NVMe devices, selects them according to defined criteria, generates a JSON description of the selected set, and exports those devices over NVMe over Fabrics while preserving their key identification attributes so that a remote initiator sees them as closely as possible to how they appear on the original local host, including correct path-awareness and ANA-related behavior in multipath environments.
