# tests/

Offline checks against the live SOCRadar API. Nothing here touches Azure.

## check_alarm_severity.py

Answers one question: what severities do real alarms actually carry, and how
big are they. Used to decide whether the analytic rules in `Analytic Rules/`
can ever fire on real data, and whether the import playbook's page size is
safe.

```
export SOCRADAR_API_KEY=...
export SOCRADAR_COMPANY_ID=...
python3 check_alarm_severity.py --days 30
```

Writes a JSON report next to the script. That report is gitignored
(`tests/*.json`) because it can contain live alarm data. The tenant the report
came from is never recorded here.

### What a 30-day preprod sample showed

- severity: the overwhelming majority of alarms are MEDIUM. HIGH and CRITICAL
  each accounted for well under one percent. They exist, they are just rare -
  so a short test window can easily contain none, and an analytic rule that
  filters on them can look broken when it is merely unexercised. Prove that
  rule against a sample long enough to contain a HIGH or CRITICAL alarm.
- size: alarms are large. The average was a few hundred KB and the largest was
  over a megabyte, which puts a single 100-alarm API page in the tens of MB.
  That is the size of one page in one HTTP action, before any accumulation
  across pages.

Re-run the script to get current numbers for your own tenant.
