# Forge the Day

A mobile accountability prototype for turning goals into explicit daily commitments, with local persistence and temporary voice capture.

**Independent project by Damiyen Lane · React Native / Expo · TypeScript · SQLite**

**Status:** unfinished private prototype. This repository is a documentation case study, not the app or an installable demo.

## Why I built it

Goals often become disconnected from daily actions. I wanted a mobile workflow where people could draft a proposal, choose what to commit to, and return to that decision later without losing state.

The prototype includes resumable onboarding, a persisted Today ledger, manual proposals, explicit confirmation, and voice recording. Recordings and transcripts do not yet generate proposals. Optional transcription is separate from the local capture path and excluded from the documented development launch configuration.

For a concrete technical example, start with [the Android recorder investigation](docs/engineering-example.md). [Architecture and evidence](docs/architecture.md) covers the persistence design and validation record.

## Engineering highlights

- **Confirmation below the UI.** Typed commands and runtime schemas distinguish proposals from commitments. Generic updates cannot bypass user confirmation or alter reviewed meaning.
- **Persistence and recovery.** Records, change events, and retry receipts commit together. Migrations, revision checks, and idempotent operations cover restarts, stale edits, and repeated taps; SQLite and in-memory adapters share contract tests.
- **Recording lifecycle debugging.** Android could pause a recorder before the app received its background event. A conditional stop then skipped finalization, allowing recording to resume while the UI said it was interrupted. The case study explains the correction, regression test, and historical device observations.
- **Privacy in the workflow.** Voice capture is temporary; remote transcription requires consent, pairing, and explicit submission. Transcript review cannot confirm a commitment, and provider credentials stay behind the backend.

## Testing and remaining work

The published evidence summarizes private CI covering type checks, migrations, rollback, retries, confirmation rules, and audio state transitions. Historical Android checks covered state preservation, recording cleanup, backgrounding, and screen lock. The tested binary predates the final dependency patch refresh; these results do not prove current iOS behavior or external acceptance.

Voice-to-proposal processing and the complete daily accountability loop remain unfinished. Current iOS validation, further Android interruption/accessibility scenarios, update preservation, and external acceptance remain open. Local records are not application-encrypted, and dependency findings remain unresolved.

## Read further

- [Architecture, technology, and evidence by scope](docs/architecture.md)
- [When an interrupted recorder kept recording](docs/engineering-example.md)

The public material describes product decisions, implementation, debugging, and testing. Application source, private builds, recordings, and user data are not published here.
