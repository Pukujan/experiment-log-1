hermes-cortex__001-teresa-scaffold-wiring
tags: hermes, cortex, windows, bash, scaffold, audit-log

target repo: C:\Users\pujan\teresa-job-search
decision: scaffold mode, keep .cortex docs shards empty until sources are added
doctor: python C:\Users\pujan\OneDrive\Desktop\web dev\webdev 2.0\collective-docs\stupidly-simple-cortex-workspace\stupidly-simple-cortex\scripts\cortex.py doctor --workspace C:\Users\pujan\teresa-job-search\.cortex
failure: bash scripts/run-pipeline.sh 0 used package workspace unless before-task accepted explicit workspace
fix: cortex.py before-task --workspace added
fix: scripts/run-pipeline.sh passes --workspace and converts bash paths to C:/ Windows paths for python.exe
failure: bash scripts/run-pipeline.sh 2 passed /mnt/c path to Windows python.exe
fix: scripts/run-pipeline.sh converts audit_logger.py path through cygpath -m or wslpath -m
failure: audit_logger.py search printed unicode arrow and crashed in PowerShell cp1252
fix: audit_logger.py prints ASCII ->
verify: bash -n scripts/run-pipeline.sh
verify: bash scripts/run-pipeline.sh 0
verify: bash scripts/run-pipeline.sh 2
verify: python scripts\audit_logger.py search --term cortex
