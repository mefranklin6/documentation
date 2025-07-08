# Overview

This document provides context for a suite of shell-based `curl` test scripts for OpenAV device microservices. These scripts are intended to facilitate lightweight, repeatable integration and regression testing of RESTful API endpoints used to control and query devices such as projectors, displays, DSPs, and cameras.

Each script issues a sequence of HTTP GET and PUT requests to simulate realistic usage patterns and verify endpoint responsiveness and correct behavior.

# Purpose

The primary purpose of these tests is to:

- Confirm endpoint availability over the network
- Validate the expected response behavior of each API endpoint
- Provide a repeatable mechanism for verifying GitHub pull requests, firmware updates, or config changes
- Allow human or automated execution of basic functional API tests with minimal dependencies

# Scope

Each test script covers the full set of GET and PUT calls supported by the associated device microservice. In most cases, the scope of the script includes:

- Power state control and verification
- Volume and mute settings
- Video input routing
- Audio mode and logic selector status
- Preset application
- Voicelift and calibration features (where applicable)

Scripts include a `sleep` delay between requests:
- `sleep 1` is inserted after each request to allow for device processing and avoid race conditions.
- `sleep 10` is used after PUT requests to the `power` endpoint, acknowledging that power state transitions may require longer device response times.

# Usage Instructions

1. **Configure constant variables**  
   Before running a script, edit the values of the constant variables at the top of the script. For example:
   ```bash
   MICROSERVICE_URL="your.microservice.domain"
   DEVICE_FQDN="device.example.local"
   INSTANCE_TAG="vol1"           # for Biamp scripts
   PRESET_ID="3"                 # if required
   ```

2. **Make the script executable**  
   ```bash
   chmod +x ./<script_name>.sh
   ```

4. **Make sure the device FQDN is reachable over a network from the microservice host**

3. **Run the script**  
   ```bash
   ./<script_name>.sh
   ```

# Limitations

- These tests do not validate the HTTP response body content; only the requests are issued.
- No authentication or session management is included; scripts assume open or IP-restricted access.
- These scripts are not substitutes for full-featured integration tests and should be used for basic verification and diagnostics.

# Scripts Currently Available

- `pjlink_curl_tests.sh`
- `visca_curl_tests.sh`
- `sony_fpd_curl_tests.sh`
- `biamp_curl_tests.sh`
- `kramer_switcher_curl_tests.sh`
- `shure_dsp_curl_tests.sh`
- `global_cache_curl_tests.sh`
- `crestron_dm_switcher_curl_tests.sh`
- `rs232_extron_curl_tests.sh`

# Recommendations

- Run these tests in a controlled environment (lab, test VLAN, or during maintenance windows).
- Consider adapting the scripts into CI/CD pipelines or cron jobs for automated health checks.
