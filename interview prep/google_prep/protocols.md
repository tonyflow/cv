## HTTPS
* http range request

### gRPC

```protocolbuffers
message UpdateRequest {

}

message UpdateResponse {

}

service Vehicle {
	rpc CheckForUpdates (UpateRequest) returns (UpdateResponse)

	rpc ReportTelemetry (TelemetryRequest) return (TelemetryResponse) 
}
```