# FestPass Jenkins challenge: evidence 

Alias (Git author): Aanya2175
Job: `festpass-ci` (Pipeline script from SCM, `git://127.0.0.1:9418/festpass-java`, branch `*/main`, script path `Jenkinsfile`, agent `student-laptop`)
Class code: 001   Challenge code: 00050

The original project documentation is in [PROJECT_README.md](PROJECT_README.md) and the task brief is in [ASSIGNMENT.md](ASSIGNMENT.md).

## Commit trace

| Commit | Message | What it did |
|---|---|---|
| 36b5a42 | Configure my Jenkins assignment | Added the `pollSCM` trigger and `submission.properties` (the first build failed on setup) |
| 2d290a4 | Add FestPass project files | Added the project files that were missing from the repo |
| 2c37a9d | Add submission codes | Filled in the class and challenge codes |
| e084af0 | Personalize my help introduction | Added my alias to the help intro (automatic red) |
| 155d39d | Fix cancel so it removes the booking | The bug fix (automatic green) |

## Red build: #5
- Cause: Started by an SCM change (no Build Now click)
- Checked-out commit: `e084af0`, which matches `git log`
- Result: FAILURE, 28 tests, 3 failures, 0 errors
- Failing assertion: `cancellationFreesSeat`: "SOLD OUT. Even the developer needs a ticket." (`BookingService.book`, line 60)
- The other two failures: `cancellationsSurviveReload` ("Cancellation was lost") and `cliWorkflowPersistsAcrossCommands` ("List does not reflect persisted cancellation")
- Skipped stages: Build JAR, Smoke test JAR, Save artifact. No JAR was published (`JarSha256` is `NOT_PUBLISHED` in the CSV)
- Evidence CSV: [evidence/red-build-5/submission-evidence.csv](evidence/red-build-5/submission-evidence.csv). It is published even though tests failed.

## Green build: #6
- Checked-out commit: `155d39d`, which matches `git log`
- Result: SUCCESS, 28 tests, 0 failures, 0 errors
- Evidence CSV: [evidence/green-build-6/submission-evidence.csv](evidence/green-build-6/submission-evidence.csv)
- JAR: [evidence/green-build-6/festpass.jar](evidence/green-build-6/festpass.jar), SHA-256 `8e9348b67867bf5d941533200898fb90d59a4ec95c1f371b9a1645c3837a71b2`, the same hash as in the CSV

Output of `java -jar festpass.jar help` (JDK 21):

```text
FestPass - campus event registration (personalized by Aanya2175)
Usage: java -jar build/festpass.jar [--data FILE] COMMAND
  add-event ID "TITLE" CAPACITY
  list
  book EVENT_ID STUDENT_ID
  cancel EVENT_ID STUDENT_ID
  roster EVENT_ID
  summary
  help
```

## What I changed and why

The failing tests were all about cancelling a booking. In `BookingService.cancel`, the code only checked that the student had a booking and then did nothing, so the student stayed in the list. That caused all three failures:

- Seats were never freed, so the next booking said SOLD OUT.
- The cancellation was never saved because the data file is written from the same list.
- The CLI `list` still counted the cancelled student.

I added one line after the check:

```java
event.students.remove(studentId);
```

No tests, pipeline stages or build helpers were changed. The only Jenkinsfile change was the polling trigger.

## Build history

| Build | Started by | Commit | Result |
|---|---|---|---|
| 1 | User (Build Now) | 36b5a42 | FAILURE in Submission setup: only `Jenkinsfile` and `submission.properties` were committed, so `ci/BuildEvidence.java` was missing |
| 2 | User (Build Now) | 2d290a4 | FAILURE in Submission setup: codes in `submission.properties` were still placeholders |
| 3 | SCM change | 2c37a9d | FAILURE in Test (3 failures), the first real red run |
| 4 | User (Build Now) | 2c37a9d | FAILURE in Test (3 failures), a manual re-run of the same commit |
| 5 | SCM change | e084af0 | FAILURE in Test (3 failures), the automatic red build used as evidence |
| 6 | SCM change | 155d39d | SUCCESS, 28 tests, 0 failures |
