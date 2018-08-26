# Reliable, Scalable and Maintainable Applications

Applications today are *data-intensive*, not *compute-intensive*.
Building Blocks include:
 - databases - storage and retreival
 - caches - remembering expensive lookups
 - search indexes - enable keyword search
 - stream processing - send message async.
 - batch processing - accumulate large amount of data.

## Reliability
 - performs functions as expected.
 - tolerate user errors.
 - good enough performance under expected load.
 - prevent unauthorized access.

### Faults
  - anticipate = fault tolerant (resiliant).
    - hardware
    - software
    - human error
      - decoupled sandbox environment
      - intigration tests
      - monitoring

## Scalability
  - cope with increased load
    - what is the **load parameter** used for metric?
      - requests/second; simulanteous reads.

    - with correct load paramter determined: **performance** measurements:
      - ++**load parameter**; keep system resources unchanged; how is performance.
      - ++**load parameter**; how much do we need to ++**resources** to keep performance unchanged.

    - % repsonse time reporting.
     - p95, p99, p999 percentile.
     - SLAs(Service-Level-Agreement
       - "Service is up if has a median response time of < 200ms and a 99th percentile under 1s".
     - response times should be measured client-side.
    
    - cope with load
     - scale-up (++CPU/RAM/etc)
     - scale-out (++machines)
     - elastic?

## Maintainability
Make system simpler to maintain.
 - Operability: easier for operations team to keep system running.
 - Simplicity: easier for new engineers to understand the system
   - removal of as much complexity as possible.
 - Evolvability: easier to make changes to the system.
