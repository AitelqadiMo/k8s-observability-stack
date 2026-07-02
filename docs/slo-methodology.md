# SLO methodology

This stack implements availability SLOs using the multi-window,
multi-burn-rate approach from the Google SRE workbook.

## Why burn rate, not a static threshold

A fixed "alert when error rate > 1%" fires on harmless blips and stays silent
on slow leaks that quietly drain the month's error budget. Burn-rate alerting
ties the signal to how fast you are spending the budget, so paging maps to real
user pain.

## The numbers

For a 99.9% availability target, the error budget is 0.1% of requests.

| Alert | Burn rate | Windows | Budget consumed | Action |
|-------|-----------|---------|-----------------|--------|
| Fast burn | 14.4x | 5m and 1h | ~2% in 1h | Page |
| Slow burn | 6x | 30m and 6h | slower leak | Ticket |

Each alert requires *both* a short and a long window to cross the threshold.
The long window confirms the problem is sustained; the short window makes the
alert recover quickly once the incident is resolved.

## Adapting to your service

1. Point the recording rules at your own request metric (the examples assume
   `http_requests_total` with a `code` label).
2. Change the `0.001` budget constant if your target is not 99.9%.
3. Add latency SLOs by recording a "good vs total" ratio from histogram
   buckets and cloning the burn-rate group.
