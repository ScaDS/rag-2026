# Changelog

## 2026-03-16

- Changed default module release on all cluster to `release/25.06`.

## 2025-11-27

- Changed default release on all cluster to `release/24.10`.

## 2025-11-26

- `Alpha Centauri` und `Capella`: changed
  [resource limits for the partitions `alpha-interactive` and `capella-interactive`](../jobs_and_resources/slurm_limits.md#qos-resource-limits)

## 2025-10-29

- [Barnard](../jobs_and_resources/hardware_overview.md#barnard): system update: Red Hat Enterprise
Linux 9.6
- [Capella](../jobs_and_resources/hardware_overview.md#capella): system update: Rocky Linux 9.6
- Lustre (/data/horse, /data/walrus): system update
- WEKAio (/data/cat): system update
- IntelliFlash (/home, /software): system update, changing network routes to improve performance
and stability
- Capella: [Changed SSH host keys](../access/key_fingerprints.md#capella)
- [Romeo](../jobs_and_resources/hardware_overview.md#romeo): [Changed SSH host keys](../access/key_fingerprints.md#romeo)

## 2025-09-04

### Changed

- [Alpha Centauri](../jobs_and_resources/hardware_overview.md#alpha-centauri):
system update: Rocky Linux: 9.6, GPU driver: 580.65.06, System CUDA: 13.0
- Alpha Centauri: `login[1,2]` are now aliases for `i8001` and `i8002`.
- Alpha Centauri: removed GPU availability on Alpha Centauri login nodes.

### Added

- Alpha Centauri: partition `alpha-interactive` containing `i8001` and `i8002`,
each running now with 16 virtual GPUs and 16 cores available only via Slurm.
- workspace filesystem `quokka`: 1.7 PB available for high IOPS rates
on Alpha Centauri (also on Romeo, Datamover and Dataport nodes);
maximum duration 30 days, 2 extensions, 30 days grace period after deletion.
