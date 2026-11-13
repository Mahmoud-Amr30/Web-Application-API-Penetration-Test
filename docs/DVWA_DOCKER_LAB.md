# DVWA Docker Compose Lab — Deployment Runbook

**Status:** NOT YET VERIFIED. This is a runbook for the assigned implementation task.

## Official source

- Repository: https://github.com/digininja/DVWA
- Official README: https://github.com/digininja/DVWA#docker
- Official Compose file: https://github.com/digininja/DVWA/blob/master/compose.yml

The official project documents Docker and Docker Compose as prerequisites. Its Compose setup brings up DVWA and MariaDB and makes DVWA available at `http://localhost:4280`. The Compose file currently publishes `127.0.0.1:4280:80`; verify the checked-out file before starting.

## A. Preflight

Run on the workstation selected by the team:

```bash
docker version
docker compose version
```

If either command fails, record the exact error. Do not install or change the container engine without the machine owner agreeing.

## B. Clone the official DVWA source separately

Keep the DVWA source separate from the team's documentation repository:

```bash
mkdir -p ~/Documents/Freelance/DEPI_Lab
cd ~/Documents/Freelance/DEPI_Lab
git clone https://github.com/digininja/DVWA.git
cd DVWA
```

Record the version/commit used:

```bash
git rev-parse HEAD
```

## C. Verify loopback-only exposure before startup

Inspect the Compose port mapping:

```bash
grep -n -A3 -B2 'ports:' compose.yml
```

Expected safe mapping from the official Compose file:

```yaml
ports:
  - 127.0.0.1:4280:80
```

If the mapping is not loopback-only, stop and review it with the team before starting. Do not expose DVWA on the LAN or Internet.

## D. Start the lab

```bash
docker compose up -d
docker compose ps
```

Then test local reachability:

```bash
curl -I http://127.0.0.1:4280
```

Open `http://127.0.0.1:4280` in a browser. Follow the DVWA setup page to initialize the database if required by the running version.

## E. Capture implementation evidence

Do not mark the task Done until the team has:

- [ ] Captured `docker version` and `docker compose version` output.
- [ ] Captured `docker compose ps` showing services running.
- [ ] Captured the browser showing the DVWA page.
- [ ] Recorded the DVWA Git commit from `git rev-parse HEAD`.
- [ ] Verified the published port is loopback-only.
- [ ] Recorded tester, date, host OS, and any deviations.
- [ ] Redacted sensitive values before adding screenshots to GitHub.

Evidence naming suggestion:

- `SCR-001-dvwa-browser.png`
- `LOG-001-docker-compose-ps.txt`
- `LOG-002-dvwa-commit.txt`
- `LOG-003-loopback-port.txt`

Place only sanitized copies in `evidence/sanitized/` in the team repository. Never copy `.env`, raw secrets, or unredacted traffic.

## F. Stop the lab

To stop services without deleting the named database volume:

```bash
docker compose stop
```

Avoid `docker compose down -v`; `-v` removes volumes and can erase lab database state.

## G. API lab caveat

The official DVWA README notes that the API lab requires Apache `mod_rewrite` and Composer vendor dependencies. Do not claim API test coverage until the team verifies these prerequisites for the selected Docker deployment. Record gaps and ask the instructor whether a supplementary authorized API lab is acceptable.
