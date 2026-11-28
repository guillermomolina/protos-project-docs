# PERF048 — Initial slow Java test evidence

Date: 2026-11-28. Owner: [PERF048 #1227](https://github.com/guillermomolina/protos/issues/1227). Trigger: DIST028 #1226. Independent top-level performance investigation; implementation not started.

| Revision | CI evidence | Class duration | Result |
| --- | --- | ---: | --- |
| a556d7c442ed9c5d9b189846704e5a203b45143e | [GraalVM 25.5](https://github.com/guillermomolina/protos/actions/runs/46106710592/job/140861523627) | 732.8 s | 9/9 PASS |
| ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa | [GraalVM 25.6 attempt 1](https://github.com/guillermomolina/protos/actions/runs/46112284172/job/140878878175) | 910.8 s | 1 failure |
| ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa | [GraalVM 25.6 attempt 2](https://github.com/guillermomolina/protos/actions/runs/46112284172/job/140883695716) | 504.9 s | 9/9 PASS |

The observations are for ProtosTestToolRepositoryScopePublicIntegrationTest, not isolated comparable benchmarks. They establish pathological existing test cost and variability, not a causal GraalVM slowdown.

The failed method hierarchicalLibraryScopeRetainsItsCompleteCorpus reached assertCompleteLibrary, then libraryRepositoryListing and runInScope("libraries", "test", "--list-cases"). The 180-second child-process timeout occurred before the complete-library listing assertion. The support redirects output to temporary files, destroys an unfinished child, and deletes the files in finally.

Exact candidate source:
- [Repository scope integration test](https://github.com/guillermomolina/protos/blob/ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa/src/test/java/com/guillermomolina/protos/cli/selection/ProtosTestToolRepositoryScopePublicIntegrationTest.java#L33-L98).
- [CLI integration support](https://github.com/guillermomolina/protos/blob/ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa/src/test/java/com/guillermomolina/protos/cli/ProtosTestToolCaseSelectionPublicIntegrationTestSupport.java#L185-L254).
- [Makefile](https://github.com/guillermomolina/protos/blob/ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa/Makefile): TEST008 telemetry is advisory, local confirmation is not run in CI, and make check does not include timing admission.
- [Slow-test policy](https://github.com/guillermomolina/protos/blob/ad7270526751cadd5c4f2ea60aa6a1e0fbe804aa/tools/java_slow_test_policy.py): normalized suspicion threshold 10 seconds.

Investigation must cover the entire Java test suite, use causal attribution and comparable conditions, preserve coverage and independence, and avoid raising timeouts, skipping tests, blanket serialization or automatic baseline updates. Review prior PERF022, PERF031, TEST008 and PLAT047 evidence before proposing remedies. Timing-admission policy changes require separate ownership and approval. No measured cause, remediation or performance target beyond the existing policy has yet been established.
