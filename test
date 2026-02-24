cd ~/aiops-lab/env
: > lab-logs/lab2-noise.log

while true; do
  # Base latency mostly healthy
  latency=$((RANDOM%600 + 50))   # 50–649ms baseline

  # Every ~20 seconds, simulate a "degradation window" for 6 seconds
  # (Vodafone-style: dependency slowdown / DB saturation / network jitter)
  if [ $(( $(date +%s) % 20 )) -lt 6 ]; then
    latency=$((RANDOM%400 + 800))  # 800–1199ms during degradation
    # during degradation, error rate is higher
    r=$((RANDOM%10))
    lvl="info"
    if [ $r -ge 5 ]; then lvl="error"; fi   # ~50% errors in degradation
  else
    # normal window: occasional errors
    r=$((RANDOM%10))
    lvl="info"
    if [ $r -ge 7 ]; then lvl="error"; fi   # ~30% errors normally
  fi

  # Burstiness: sometimes write multiple logs in 1 second (traffic spikes)
  burst=$((RANDOM%3 + 1))  # 1–3 lines/sec
  for i in $(seq 1 $burst); do
    echo "$(date -Iseconds) service=checkout level=$lvl latency_ms=$latency" >> lab-logs/lab2-noise.log
  done

  sleep 1
done

Use these exact alert thresholds (so it behaves as promised)
A — Noisy (Bad)

Query:

sum(rate({job="labfiles"} |= "level=error"[1m]))

Alert threshold: > 0.20
For: 0s (instant spam)

B — Tuned (Good)

Query:

sum(rate({job="labfiles"} |= "level=error"[5m]))

Alert threshold: > 0.35
For: 1m (reduces flaps)

C — Latency-aware (Best)

Query:

sum(rate(({job="labfiles"} |= "level=error" |~ "latency_ms=8[0-9]{2}|latency_ms=9[0-9]{2}|latency_ms=1[0-1][0-9]{2}")[1m]))

Alert threshold: > 0.05
For: 30s–1m

Note: I included 1000–1199ms too via latency_ms=1[0-1][0-9]{2} so “Best” definitely triggers sometimes.
