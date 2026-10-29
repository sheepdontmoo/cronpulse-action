# CronPulse heartbeat for GitHub Actions

Scheduled workflows fail quietly. GitHub disables `schedule:` triggers in public repos after 60 days without activity, runners get stuck, and a failed nightly job can go unnoticed for weeks. This action sends a heartbeat to [CronPulse](https://cronpulse.wouldairecommend.workers.dev/?utm_source=github&utm_medium=action&utm_campaign=cronpulse_action) after each run, and CronPulse alerts you (email, Slack, Discord, Telegram or webhook) when a run fails **or when it never arrives at all**.

## Usage

1. Create a free check at [cronpulse.wouldairecommend.workers.dev](https://cronpulse.wouldairecommend.workers.dev/?utm_source=github&utm_medium=action&utm_campaign=cronpulse_action) with the same schedule as your workflow, and copy its private ping URL.
2. Save it as a repository secret named `CRONPULSE_PING_URL`.
3. Add the steps below to your scheduled workflow.

```yaml
on:
  schedule:
    - cron: "0 3 * * *"

jobs:
  nightly:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: sheepdontmoo/cronpulse-action@v1
        with:
          ping-url: ${{ secrets.CRONPULSE_PING_URL }}
          status: start
      - name: Run the job
        id: job
        run: ./scripts/job.sh
      - name: Report success
        if: ${{ always() && steps.job.outcome == 'success' }}
        uses: sheepdontmoo/cronpulse-action@v1
        with:
          ping-url: ${{ secrets.CRONPULSE_PING_URL }}
      - name: Report failure
        if: ${{ always() && steps.job.outcome == 'failure' }}
        uses: sheepdontmoo/cronpulse-action@v1
        with:
          ping-url: ${{ secrets.CRONPULSE_PING_URL }}
          status: fail
```

## Inputs

| input | required | default | description |
|---|---|---|---|
| `ping-url` | yes | | Private CronPulse ping URL. Keep it in a secret. |
| `status` | no | `success` | `start`, `success` or `fail` |

Requires bash and curl (present on GitHub-hosted Linux runners). Inputs are passed through environment variables, never interpolated into shell code. Curl retries three times with a ten-second timeout per attempt. Add `continue-on-error: true` if a monitoring delivery error should not fail your workflow.

## Pricing

CronPulse has a free plan. [Lifetime Pro is €69 once](https://cronpulse.wouldairecommend.workers.dev/lifetime?utm_source=github&utm_medium=action&utm_campaign=cronpulse_action) for the first 100 users, no subscription.

Made by SupaDesign ([getsupadesign.com](https://getsupadesign.com)). MIT licensed.
