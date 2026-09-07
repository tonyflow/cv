# Context: Google Automotive Staff-Level (L6+) System Design Interview
# Topic: Challenge 1: Over-the-Air (OTA) Update System (Secure, Delta Deployments with Auto-Rollback)

## 1. Problem Framing, Scope & Scale
- **System Boundaries:** End-to-end orchestration from Supply Chain Ingress (Signed Binaries) to Cloud Control Plane, down via mTLS/CDN to the In-Vehicle Gateway, and onto isolated ECUs over CAN/Automotive Ethernet using UDS (ISO 14229).
- **Core Objectives:** Secure cryptographic transport, binary delta generation/patching at edge, atomic DAG-based rollouts, dual-bank (A/B) memory execution, and zero-trust verification.
- **Scale Estimations:** 
  - Fleet Size: 10,000,000 vehicles.
  - Comms Pattern: 15-minute polling interval to protect 12V vehicle battery and eliminate keep-alive cell costs.
  - Peak Request Throughput: ~4,630 QPS at the Gateway API during thundering herd rush hours.
  - Monthly Data Egress: ~1 Petabyte/month (calculated on progressive 100MB delta rollouts over 7 days).

## 2. API Contract & Protocols
- **Control Plane & Telemetry:** Managed via gRPC over HTTP/2 for small, rigid, binary-serialized payloads.
- **File Distribution:** Managed via Raw HTTP/2 Byte-Range Requests directly to the CDN Edge Cache to offload heavy network traffic from the core transactional backend database.

### Protobuf Service Schema (`ota_service.proto`)
```protobuf
syntax = "proto3";
package google.automotive.ota.v1;

service VehicleOTAService {
  rpc CheckForUpdates (UpdateRequest) returns (UpdateResponse);
  rpc ReportTelemetry (TelemetryRequest) returns (TelemetryResponse);
}

message UpdateRequest {
  string vin = 1;
  string model_id = 2;
  string region = 3;
  map<string, string> current_ecu_versions = 4; 
}

message UpdateResponse {
  bool update_available = 1;
  string campaign_id = 2;
  repeated ECUUpdateTask tasks = 3;
}

message ECUUpdateTask {
  string ecu_id = 1;
  string target_version = 2;
  string binary_url = 3;         // Pre-signed CDN URL with short TTL
  string binary_sha256 = 4;      
  string signature = 5;          
  int64 binary_size_bytes = 6;
  bool is_delta = 7;             
}

message TelemetryRequest {
  string vin = 1;
  string campaign_id = 2;
  string execution_state = 3;    // DOWNLOADING, VERIFYING, FLASHING, COMPLETED, ROLLBACK_TRIGGERED
  string error_code = 4;         
  string details = 5;
}

message TelemetryResponse {
  bool acknowledge = 1;
}
```

## 3. High-Level Architecture Components
1. **Ingress Plane:** Release Management UI + Async Delta Generation Workers (bspatch/hdiff).
2. **Cloud Control Plane:** Edge Gateway API (mTLS authentication) + Campaign & Target Resolution Engine + State & Metadata Store (Google Cloud Spanner for strong global consistency on VIN software states/SBOMs).
3. **Distribution Layer:** Global CDN Edge Cache serving chunked file segments via HTTP 206 Partial Content codes.
4. **Vehicle Edge Plane:** Vehicle OTA Orchestrator (manages downloads, resumption byte positions, and Uptane safety metadata verification) + In-Vehicle Flash Bridge (UDS engine interacting directly with physical Bank A / Bank B split memory layouts on target microcontrollers).

## 4. Deep-Dive Core Optimizations
- **Security (Uptane Framework):** Separation of duties via an online Director Key (managed by the cloud for targeting validation) and an offline Image Signer Key (isolated in an HSM during compilation). The edge orchestrator enforces an invariant that rejects any update lacking both valid signatures, neutralizing compromised cloud plane attacks.
- **Network Resiliency (HTTP Range Requests):** When an edge download resumes (e.g., exiting a tunnel), the car fetches a new short-TTL pre-signed URL from the Gateway API, then hits the CDN with a `Range: bytes=X-` header, prompting an HTTP 206 response to save cellular costs and memory overhead.
- **Binary Delta Mechanics:** Delta Generation workers calculate differences between Operand 1 (Old Binary Base) and Operand 2 (New Target Image), outputting a highly compressed instruction patch file (COPY/ADD/EXTRA). The edge reassembles the standalone cohesive unit in background scratch space block-by-block (`SHA-256` validated) without wearing down active flash memory blocks.
- **Dependency Orchestration:** Updates are mapped as a Directed Acyclic Graph (DAG) sorted topologically. A safe-state gated commit locks the physical A/B switchover until *all* dependent ECUs pass their post-flash checksums; a single module failure triggers an atomic all-or-nothing rollback.

## 5. Operations & Production Rollout
- **Canary Strategy:** Concentric rings split across Ring 0 (Internal Testing Fleet / 5k cars / 48h) -> Ring 1 (Opt-in Beta Users / 50k cars / 5 days) -> Ring 2 (Regional Canary / 500k cars / 7 days) -> Ring 3 (Global Production / 9.5M+ cars / 14-day phased expansion).
- **Automated Tripwire Flags:** Production campaigns automatically suspend globally if telemetry reports:
  1. Delta block assembly misalignment failure rates > 0.1%
  2. UDS flash write protocol abort rates > 0.05%
  3. Hardware watchdog rollbacks (Bank B reverting back to Bank A) > 0.01%
- **Configuration Format State:** The vehicle's active Software Bill of Materials (SBOM) is tracked locally on an embedded SQLite partition and synced globally to Google Cloud Spanner. Schema migrations leverage a dual-write pipeline via Kafka event streams to isolate and preserve legacy client backward compatibility.
