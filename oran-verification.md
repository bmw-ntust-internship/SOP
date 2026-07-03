<h1 align="center">O-RAN Interface Verification — O1 / E2 / R1 + PM Pipeline</h1>

---

> [!IMPORTANT]
> These are the **lab-common conformance checks** for every O-RAN project testbed. They are required in two places:
>
> - at the end of every project's [implementation.md](./implementation.md) (Post-Installation Verification), and
> - for the **D-180 code & project review** before the oral exam ([leaving-procedure.md Section 4](./leaving-procedure.md#4-oral-examination-requirements)): show O1 / E2 / R1 test conformance — data collection through O1/E2, configuration through O1/E2, and R1 for SMO-internal component communication.

Each check follows the SOP command-block convention: the command and its expected output in one block, output lines prefixed with `#`.

1. **Verify O1 Interface (NETCONF/YANG + VES):**

   The O1 interface connects the SMO/Non-RT RIC to O-RAN Network Functions (O-DU, O-CU) using:
   - **NETCONF** (port 830) for CM (Configuration Management)
   - **VES** (`POST /eventListener/v7`) for FM (Fault Management) and PM (Performance Management) event streaming

   > [!NOTE]
   > Reference: [O-RAN SC OAM Project](https://docs.o-ran-sc.org/projects/o-ran-sc-oam/en/latest/) and O-RAN.WG10.O1-Interface specification.

   1. **Verify NETCONF (CM) — O-RAN NF side:**

      ```bash
      # Check NETCONF server is listening on port 830 on the O-RAN NF
      netstat -tnlp | grep 830

      # tcp  0  0  0.0.0.0:830  0.0.0.0:*  LISTEN  12345/netconfd
      ```

      ```bash
      # Open a NETCONF session and retrieve YANG capabilities from the NF
      ssh -p 830 -s netconf admin@<O-RAN-NF-IP>

      # <?xml version="1.0" encoding="UTF-8"?>
      # <hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
      #   <capabilities>
      #     <capability>urn:ietf:params:netconf:base:1.1</capability>
      #     <capability>urn:o-ran:o1:1.0</capability>
      #     <capability>urn:o-ran:supervision:1.0</capability>
      #     ...
      #   </capabilities>
      # </hello>
      ```

   2. **Verify VES Collector (FM/PM) — SMO side:**

      The VES Collector receives HTTP POST events on `/eventListener/v7`. Do **not** use `GET /events` — that endpoint does not exist. Verify by inspecting container logs.

      ```bash
      # Tail VES collector container logs to confirm incoming events
      kubectl logs -n onap deployment/dcae-ves-collector --tail=50 | grep -E "pnfRegistration|heartbeat|stndDefined|Fault"
      # (or for Docker: docker logs ves-collector --tail=50)

      # [INFO] VES event received: domain=pnfRegistration, sourceName=o-ran-nf-1
      # [INFO] VES event received: domain=heartbeat, sourceName=o-ran-nf-1, sequence=1
      # [INFO] VES event received: domain=stndDefined, stndDefinedNamespace=3GPP-PerformanceAssurance
      ```

      ```bash
      # Optionally: send a test VES event manually to confirm the collector is reachable
      curl -X POST http://<VES-collector-IP>:8080/eventListener/v7 \
        -H "Content-Type: application/json" \
        -u <user>:<password> \
        -d '{"event":{"commonEventHeader":{"domain":"heartbeat","eventName":"heartbeat_O_RAN_COMPONENT","sourceName":"test-nf","version":"4.0","vesEventListenerVersion":"7.2"}}}'

      # HTTP/1.1 202 Accepted
      ```

2. **Verify E2 Interface (E2AP over SCTP):**

   The E2 interface connects the Near-RT RIC to E2 Nodes (O-CU-CP, O-CU-UP, O-DU) using E2AP over SCTP.

   ```bash
   # Confirm SCTP association is established between Near-RT RIC and E2 Node
   ss -tn | grep 36421

   # ESTAB  0  0  <Near-RT-RIC-IP>:36421  <E2-Node-IP>:XXXXX
   ```

   ```bash
   # Check Near-RT RIC logs for a successful E2 Setup procedure
   grep -i "E2 Setup Response\|E2SetupResponse" /var/log/near-rt-ric/*.log

   # [INFO]  Received E2SetupResponse from E2-Node: o-du-1 (PLMN: 00101, NB-ID: 1)
   # [INFO]  E2 interface to o-du-1 is UP
   ```

   ```bash
   # Verify an xApp subscription is active and RIC Indications are flowing
   grep -i "RICIndication\|Indication received" /var/log/xapp/*.log | tail -5

   # [INFO]  RICIndication received from E2-Node: o-du-1, RAN Function: 2, Sequence: 1041
   # [INFO]  RICIndication received from E2-Node: o-du-1, RAN Function: 2, Sequence: 1042
   ```

3. **Verify R1 Interface (rApp ↔ Non-RT RIC via DME & SME):**

   The R1 interface is the service-based interface between rApps and the Non-RT RIC platform (O-RAN.WG2.R1AP specification). It is composed of two functional blocks:

   - **DME (Data Management and Exposure)** — realized by the **Information Coordination Service (ICS)**. Decouples data producers from data consumers via Information Types and Information Jobs.
   - **SME (Service Management and Exposure)** — realized by the **CAPIF-based Service Manager** (3GPP CAPIF APIs). Enables rApps and SMO components to register, publish, discover, and invoke each other's service APIs through a Kong API gateway.

   > [!NOTE]
   > References:
   > - [O-RAN SC ICS (DME)](https://docs.o-ran-sc.org/projects/o-ran-sc-nonrtric-plt-informationcoordinatorservice/en/latest/)
   > - [O-RAN SC SME (CAPIF)](https://docs.o-ran-sc.org/projects/o-ran-sc-nonrtric-plt-sme/en/latest/)

   1. **Verify DME (Information Coordination Service / ICS):**

      ```bash
      # Check ICS (DME) is running and healthy
      curl -s http://<ICS-IP>:<port>/status

      # {"status":"success"}
      ```

      ```bash
      # List all registered Information Types (data types exposed over R1)
      curl -s http://<ICS-IP>:<port>/data-producer/v1/info-types | python3 -m json.tool

      # [
      #   "pm-data-type-1",
      #   "fault-event-type-1"
      # ]
      ```

      ```bash
      # List all registered data producers
      curl -s http://<ICS-IP>:<port>/data-producer/v1/info-producers | python3 -m json.tool

      # [
      #   "producer-rapp-1",
      #   "producer-rapp-2"
      # ]
      ```

      ```bash
      # List active Information Jobs (data consumer subscriptions)
      curl -s http://<ICS-IP>:<port>/data-consumer/v1/info-jobs | python3 -m json.tool

      # [
      #   {
      #     "info_job_id": "job-001",
      #     "info_type_id": "pm-data-type-1",
      #     "job_owner": "consumer-rapp-1",
      #     "status": "ENABLED"
      #   }
      # ]
      ```

   2. **Verify SME (Service Management and Exposure / CAPIF):**

      SME uses the CAPIF core function to let API providers (rApps, SMO components) publish services, and API invokers (rApps, consumers) discover and call them via the Kong gateway.

      ```bash
      # Check the SME / CAPIF core function is running
      curl -s http://<SME-IP>:<port>/api-provider-management/v1/health

      # {"status":"OK"}
      ```

      ```bash
      # List all published service APIs (APIs registered by providers/rApps)
      curl -s http://<SME-IP>:<port>/published-apis/v1/<apfId>/service-apis | python3 -m json.tool

      # [
      #   {
      #     "apiId": "api-001",
      #     "apiName": "pm-kpi-rapp-api",
      #     "description": "Exposes PM KPIs from rApp over R1",
      #     "aefProfiles": [{"aefId": "aef-001", "versions": [{"apiVersion": "v1"}]}]
      #   }
      # ]
      ```

      ```bash
      # Discover available service APIs as an invoker (rApp or SMO component)
      curl -s "http://<SME-IP>:<port>/service-apis/v1/allServiceAPIs?api-invoker-id=<invoker-id>" \
        | python3 -m json.tool

      # {
      #   "serviceAPIDescriptions": [
      #     {
      #       "apiName": "pm-kpi-rapp-api",
      #       "apiId": "api-001",
      #       "aefProfiles": [{"versions": [{"apiVersion": "v1"}], "protocol": "HTTP_1_1"}]
      #     }
      #   ]
      # }
      ```

      ```bash
      # Verify Kong gateway is routing requests to a registered API
      curl -s http://<Kong-gateway-IP>:<port>/<api-route>/v1/health

      # {"status":"OK"}
      ```

4. **Verify InfluxDB — PM Data Stored via O1:**

   PM events received over O1 (VES `stndDefined` / file-based PM) are parsed and stored in InfluxDB by the SMO's PM data pipeline.

   ```bash
   # Check InfluxDB is running and reachable
   curl -s http://<InfluxDB-IP>:8086/health

   # {"checks":[],"commit":"...","date":"...","name":"influxdb","status":"pass","version":"2.x.x"}
   ```

   ```bash
   # Query the latest PM measurements written from O-RAN NF (adjust bucket/measurement names)
   influx query '
     from(bucket: "oranpm")
       |> range(start: -1h)
       |> filter(fn: (r) => r._measurement == "NRCellDU")
       |> last()
   ' --host http://<InfluxDB-IP>:8086 --token <your-token> --org <your-org>

   # Result: _measurement  _field         _value  sourceName    _time
   #         NRCellDU      RRU.PrbUsedDl  72.5    o-du-1        2024-10-21T10:45:00Z
   #         NRCellDU      RRU.PrbUsedUl  58.1    o-du-1        2024-10-21T10:45:00Z
   ```

   ```bash
   # (Alternative) Query via InfluxDB HTTP API with Flux
   curl -s -X POST http://<InfluxDB-IP>:8086/api/v2/query \
     -H "Authorization: Token <your-token>" \
     -H "Content-Type: application/vnd.flux" \
     -d 'from(bucket:"oranpm") |> range(start:-1h) |> filter(fn:(r) => r._measurement == "NRCellDU") |> last()'

   # ,result,table,_start,...,_measurement,_field,_value,sourceName
   # ,_result,0,...,NRCellDU,RRU.PrbUsedDl,72.5,o-du-1
   ```

5. **Verify Grafana — PM Data Displayed from InfluxDB via R1:**

   Grafana uses InfluxDB as a datasource. In an O-RAN setup, the data consumed by Grafana dashboards comes from InfluxDB, which is populated via O1 PM. An rApp may expose aggregated KPIs over the R1 interface, which are also stored in InfluxDB for visualization.

   ```bash
   # Check Grafana is running and healthy
   curl -s http://<Grafana-IP>:3000/api/health

   # {"commit":"...","database":"ok","timestamp":"...","version":"10.x.x"}
   ```

   ```bash
   # Verify the InfluxDB datasource is configured and reachable in Grafana
   curl -s -u admin:<grafana-password> http://<Grafana-IP>:3000/api/datasources | \
     python3 -m json.tool | grep -E '"name"|"type"|"url"'

   # "name": "InfluxDB-ORANPM",
   # "type": "influxdb",
   # "url": "http://<InfluxDB-IP>:8086",
   ```

   ```bash
   # Test the datasource connectivity from Grafana to InfluxDB
   curl -s -u admin:<grafana-password> \
     -X POST http://<Grafana-IP>:3000/api/datasources/proxy/<datasource-id>/api/v2/query \
     -H "Content-Type: application/vnd.flux" \
     -d 'from(bucket:"oranpm") |> range(start:-5m) |> count()'

   # ,result,table,...,_value
   # ,_result,0,...,42
   ```

   > [!TIP]
   > To visually confirm: open the Grafana dashboard at `http://<Grafana-IP>:3000`, navigate to the O-RAN PM dashboard, and verify that graphs for KPIs such as PRB utilization, throughput, and cell availability show live or recent data points.

