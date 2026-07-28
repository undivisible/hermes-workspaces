# hermes-workspaces

Safe, explicit workspace selection for Hermes profiles.

A workspace is a **named Hermes profile** with its own configuration, model/tool
preset, memory, skills, session database, and gateway route. It is not a
label applied to a shared conversation.

## Design contract

`/workspace <name>` selects a configured profile for the inbound chat/topic,
then begins a new session boundary. It never changes an in-flight agent's
profile or reuses a different workspace's session. That preserves prompt-cache
integrity and prevents memory/session crossover.

Examples:

```text
/workspace 2
/workspace school
/workspace status
```

The profile aliases `2` and `school` must resolve to profiles created by
Hermes' native profile manager. Each selected source is served by
`gateway.multiplex_profiles` and a profile-specific `HERMES_HOME`.

## Security invariants

- Names resolve only against configured, existing profiles.
- A chat/topic selection is scoped to its exact platform, chat, and thread.
- Selection resets the old session before the next model turn.
- Unknown profiles fail closed; the active/default profile is not silently
  changed.
- Toolsets, memory, session DB, configuration, and secrets remain profile
  scoped.
- Phone/Telegram identity alone never grants a higher-trust workspace.

## Implementation status

The repository rename from `hermes-workspace2` to `hermes-workspaces` is
complete.

Hermes already provides the required isolated-profile runtime and static
route resolver. The remaining upstream integration is a source-aware gateway
command handler: plugin slash-command handlers receive only raw arguments, so
they cannot safely persist a selection for the originating chat/topic.
`/workspace` therefore belongs at the Hermes gateway dispatch boundary, where
it can receive `MessageEvent.source`, update the source-scoped route override,
and reset the session atomically.

No configuration secrets belong in this repository. Use Hermes' native profile
and gateway configuration surfaces for profile creation and routing.
