# Gap log — tools and gaps found while running the pipeline

| Step | Tool / task | Worked at HD scale? | Time / memory | Gap or note |
|---|---|---|---|---|
| 01 | scanpy read_10x_h5 (8 µm, 446k bins) | yes | <1 min, ~5 GB | fine |
| 01 | tar extraction of binned_outputs | yes | 6 min for 23 GB | hard links to 002um images break when extracting 008um only |
| 01 | off-tissue background / edge decay | custom code | seconds | no existing tool computes this for HD; spillover ~30-50 µm from edges |
