# Jahia Maven validation

This branch builds the public GraphQL Core module on standard Ubuntu 24.04
GitHub runners with Java 11 and Maven 3.9.16. The command retains the upstream
`clean install -DskipTests=false` build. A separate settings file uses Jahia's
public Nexus group; it does not use the private Nexus credentials or warmed
container used by upstream CI. Results describe this public hosted workload.

The workflow uses the released `boringcache/one` Action pinned to its distribution
commit. It obtains short-lived credentials through GitHub OIDC. Run its `connect`
option once and approve the exact repository, validation branch, and workflow
for the `boringcache/jahia-validation` workspace. Ordinary runs need no secret.

Run `cold` once on the base commit. This builds an uncached baseline, a remote
Maven build-cache case, and a dependency-archive case. Each BoringCache case
then runs once on a fresh runner with restore-only access. Maven dependencies
are not archived in the remote build-cache case. The dependency case archives
only `.m2/repository` and rebuilds the project. A successful build alone is not
proof that the remote cache was reused; inspect its emitted evidence and logs.

The selected upstream commits, in order, are:

| Step | Upstream commit |
| --- | --- |
| Base | c309b6f66c958091ed13bf8ae525131e091c76cc |
| 1 | 9d200d48e0d7adc711ef57820fb4dfcc459f76f2 |
| 2 | 6bb6c1bd9509779f44cebf92ddb3992b047aa3e6 |
| 3 | f4c54d3e70e687d709e3f8d089a79320451dcd9c |
| 4 | 70804ecfa1a2db4fcc6c24bf2bb1c09e6773f6ba |
| 5 | d9b15234120555136ae675f5d3f54151f31c04bb |

Apply each upstream first-parent change in order with `git cherry-pick --no-commit`
(add `-m 1` for a merge commit). Update `.github/boringcache-source` to that
original upstream SHA and commit both changes together with a signed commit.
This preserves the organization's linear-history requirement. Verify the source
matches the recorded upstream tree outside the validation files, then dispatch
`phase=commit` and wait for it to finish before advancing again.
Keep both cache tags unchanged throughout the sequence. An interrupted or
failed cold seed must be investigated before calling a later run warm.

Retain the workflow run, validation commit, source commit, tool versions,
Maven build timing, and product-emitted Action evidence. The BoringCache run
view supplies cache-side reuse and storage evidence, including new bytes written
where available. Missing measurements stay missing. Action post-step publication
is included in full job timing; the pre-post artifact may not yet contain it.

This proof does not reproduce `jahia-private`, private integration tests, or
cross-repository deduplication. The original image repository is forked at
`boringcache/jahia-docker-mvn-cache`; its full warmup needs Jahia's private source.
Do not translate a public module result into a storage-savings claim for that
private build or the proposed three-repository comparison.
