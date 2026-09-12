# Phase error report

Use one report for every failed audit or debugging session. Do not replace this structure with “fixed”.

## Phase

`<phase code>`

## Symptom

What was observed, including the exact command and relevant output.

## Reproduction

Exact prerequisites and steps to reproduce the failure.

## Expected / actual

- Expected:
- Actual:

## Root cause

Evidence-backed cause, including the failing boundary or invariant.

## Files touched

List every changed file and why it was in scope.

## Fix

Smallest root-cause fix and any design decision.

## Regression tests

Commands and outputs proving the regression is covered and the phase audit is green.

## Residual risk

Known limitations, owner and upgrade threshold; link to `DECISIONS.md` when accepted.
