# The microscope plugin's eval recording

The recording the microscope plugin's eval mocks were taken from
(`jeffrey-claude-plugin/evals/` in [jeffrey](https://github.com/petrbouda/jeffrey)), kept here so
the live SQL cases (`jeffrey-claude-plugin/evals-live/`) can run on any machine. They query a real
Jeffrey that holds these three profiles, and their graders name figures this recording holds.

| File | What it is |
|---|---|
| `checkout-baseline.jfr` | 30 s of the eval workload (`evals/fixtures/workload/`), `baseline` mode, one checkout in 50 traced as a `POST /checkout` exchange with its spans |
| `checkout-candidate.jfr` | the same workload in `candidate` mode, with its deliberate pricing regression |
| `checkout.hprof.gz` | the heap dump taken at the end of the candidate run, gzipped |

Recorded with plugin 1.0.28 by `build/scripts/record-plugin-eval-mocks.py`, JDK 25, `settings=profile`
with every exception throw, and jeffrey-events and jeffrey-tracing on the workload's class path. Figures:
5,822 `Checkout$Order` objects, 1,441 of them with a `SPRING-` coupon. sha256 of the files as the
recorder wrote them (the dump unpacked):

```
dd35c8532fec0a289286211ca8ad0dad09284933c3c5619f900ee92682a1c7fc  checkout-baseline.jfr
dc63e2d4f3e6fd0bb5a053e0c280605833b528c28164e4f8849090a4be16a0de  checkout-candidate.jfr
e5e6d03e0f4dafbaa67192943e1edb20ab02de7a412289c175b2ccd54718ad04  checkout.hprof
```

Load them into an empty Jeffrey on the plugin's endpoint, from a jeffrey checkout:

```bash
git clone --depth 1 https://github.com/petrbouda/jeffrey-recordings /tmp/jeffrey-recordings
python3 build/scripts/record-plugin-eval-mocks.py --url http://localhost:8585/api/mcp \
  --work-dir /tmp/jeffrey-recordings/plugin-evals --load
```

`--load` unpacks the dump, checks the three checksums against `recorded-profiles.json` and the
figures against what Jeffrey answers, and refuses the files when either differs. A re-recording
replaces all three files here, together with the mocks and graders it rewrites in jeffrey.
