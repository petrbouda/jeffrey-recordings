# The microscope plugin's eval recording

The recording the microscope plugin's eval mocks were taken from
(`jeffrey-claude-plugin/evals/` in [jeffrey](https://github.com/petrbouda/jeffrey)), kept here so
the live SQL cases (`jeffrey-claude-plugin/evals-live/`) can run on any machine. They query a real
Jeffrey that holds these three profiles, and their graders name figures this recording holds.

| File | What it is |
|---|---|
| `checkout-baseline.jfr` | 30 s of the eval workload (`evals/fixtures/workload/`), `baseline` mode |
| `checkout-candidate.jfr` | the same workload in `candidate` mode, with its deliberate pricing regression |
| `checkout.hprof.gz` | the heap dump taken at the end of the candidate run, gzipped |

Recorded with plugin 1.0.17 (jeffrey #378) by `build/scripts/record-plugin-eval-mocks.py`, JDK 25
and `settings=profile` with every exception throw. Figures: 5,775 `Checkout$Order` objects, 1,427 of
them with a `SPRING-` coupon. sha256 of the files as the recorder wrote them (the dump unpacked):

```
136e632e83fc5603dd6880c8be0919f41ea03a8628df953df0bfe9037ac0e941  checkout-baseline.jfr
7f72eed3ce87e00a1ab215defbe00807a6b97d1c005d4dd52f802cdd927e8c74  checkout-candidate.jfr
a484be60bdada2925afb6aab41477eb26013c9738c7a8b3f45baf305b7b5acd8  checkout.hprof
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
