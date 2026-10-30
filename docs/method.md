# Method

Every exercise follows the same four steps:

1. **Measure** the starting state with a command whose output can be compared later.
2. **Change** one thing.
3. **Test** with the same measurement and, where it makes sense, an independent one (for example the live log).
4. **Roll back**, or document what stays changed.

## Evidence rules

- Evidence is command output and log lines, shown as text and trimmed to what matters. Trimming is stated where it happens.
- Every table cell that says "works" or "blocked" refers to a measurement in the same exercise.
- What was not tested is listed in the "Limits" section of each exercise.
- Log entries are shown as time offsets, not as absolute times.

## Safety rules for the lab

- Keep a second way into the firewall open (web interface in a browser tab) while changing SSH settings.
- Test the new access path before closing the old one.
- Change rules one at a time and apply them explicitly; saving does not activate a rule.
- Skip tests that can lock out the only workstation. The brute-force lockout test was left out for this reason.
- Roll back test rules after measuring.

## What this repository does not claim

- It does not claim that these three changes make a firewall secure. They reduce specific exposures and show how to prove it.
- It does not cover production hardening, patching, logging to a central system or monitoring.
