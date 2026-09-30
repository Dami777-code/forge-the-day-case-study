# When an interrupted recorder kept recording

The most useful debugging lesson in Forge the Day came from checking native behavior rather than trusting the screen.

## Symptom

During an Android SDK 57 development-build regression, backgrounding the app produced a visible **Recording interrupted** state. Returning to the foreground could nevertheless resume the native recorder. The temporary audio file grew and Android still reported the microphone in use.

This was a concrete device defect, not simply an untested edge case.

## Cause

Expo Audio's native Android lifecycle handling paused the recorder before React Native delivered the background `AppState` event. Finalization checked `isRecording` before calling `stop()`. In the already-paused intermediate state that flag was false, so the app skipped the stop while presenting an interruption to the user. The native lifecycle could then resume the paused recording on return.

The app's interpretation of a status flag was weaker than its intended invariant: once a capture is finalized as interrupted, it must not resume recording.

## Correction

Finalization now calls `stop()` even if native state has already become paused, then inspects the finalized temporary file and produces the interruption state. The existing failure path schedules cleanup if finalization fails. Duration observed before stopping is preserved because native status can reset afterward.

Conceptually, the change was:

```text
Before:
  if native status says "recording": stop recorder
  show finalized/interrupted state

After:
  stop recorder, including the native-paused intermediate state
  inspect finalized audio and preserve observed duration
  show interrupted state, or handle finalization failure and cleanup
```

This is explanatory pseudocode newly written for the case study, not copied application source.

## Validation

**Automated regression:** the test starts a capture, changes the mocked native status to `isRecording = false` while retaining elapsed duration, and delivers a background event. It requires one stop call and an interrupted application state. This models the ordering that the previous check missed. The test is present in current source and passed in the latest main-branch CI.

**Historical physical-device check:** on the corrected Android development APK, the recording finalized after backgrounding, stayed at the same file size across a later observation, and Android no longer marked recording permission as actively used. Returning showed the interruption state. Deletion left temporary audio storage empty. Screen lock also stopped recording in the documented run.

**Limits:** that native binary predates the final Expo patch refresh. It is evidence for the tested correction, not a fresh run of current main or proof of iOS behavior. True-offline operation, phone-call/competing-media/audio-route interruption, low-storage recovery, and human TalkBack narration remain unproven for the SDK 57 lane. Earlier platform evidence cannot silently fill those gaps.

## Why it matters

A privacy-facing audio state must agree with what the operating system is doing. A convincing UI and passing happy-path test would have missed this defect. The investigation combined a reproducible state-ordering hypothesis, a small adapter correction, a focused automated test, and direct native microphone/file observations.

The scope stayed narrow: repair the interruption boundary and retain its evidence. It did not become a reason to add background recording or broaden the product.
