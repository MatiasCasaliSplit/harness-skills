# Stop conditions - option tables and rationale

Referenced from SKILL.md Steps 1, 3, 4, 5, 6. Each of these is a point where a
plausible-sounding guess is available but wrong often enough that the skill
requires presenting real options and waiting for a pick instead.

## Step 1: vague request against existing metrics

If the request is vague (e.g. "track checkouts" with no aggregation
specified), don't silently pick one existing metric or attribute
combination yourself. Turn the Step 1 search result into a concrete option
table and let the user pick - one decision, one turn:

| Option | Shape |
|--------|-------|
| `<existing metric name>` | `<aggregation>` + `<spread>` on `<event>` (reuse this) |
| `<existing metric name>` | `<aggregation>` + `<spread>` on `<event>` (reuse this) |
| `new` | Define a new metric with a different shape |

"I searched and picked the most relevant one" is not a substitute for
showing the user what was found - a search result is not itself a decision.

## Step 3: event not in the resolved list

If the event the user wants to measure isn't in the `fme_event_type` list,
don't invent an ID and don't silently substitute the closest-looking real
event either - both are guessing on the user's behalf. Present the real
event list as options:

| Option | Event |
|--------|-------|
| `<real event 1>` | Use this event instead |
| `<real event 2>` | Use this event instead |
| `not_listed` | The event exists but hasn't fired in the last 30 days (confirm exact spelling) |
| `instrument` | Doesn't exist yet - hand off to `/instrument-metric` first |

If they pick `instrument`, creating the metric now is *allowed* (the
backend does not validate `eventTypeId` existence by design) but the
metric will silently never compute until the event flows - make sure the
user understands that tradeoff if they want to proceed anyway.

## Step 4: PER vs ACROSS

If the user didn't say `PER` or `ACROSS`, present both options (with the
meaning for their chosen `aggregation`) and wait for a pick before drafting
Step 7:

| Option | Meaning | Example (`RATE`) |
|--------|---------|------------------|
| `PER` (recommended for experiments) | Computed per unit, then compared across treatments; significance-tested | Percent of users who completed checkout |
| `ACROSS` | One aggregate over the whole treatment; no significance test in experiment results | Count of unique users who completed checkout |

`PER` and `ACROSS` produce different numbers from the same events, so don't
pick silently even when the label ("conversion rate") points to `PER` -
state the recommendation and let the user confirm. The backend defaults
`spread` to `PER` when omitted; always send it explicitly.

## Step 5: missing or invalid owner

"No owner specified" in the request is not permission to pick one
yourself - it means this decision hasn't been made yet. Ask for a `USER`
email or `GROUP` name before drafting Step 7, the same way a missing event
or aggregation choice would stop you. Filling in a plausible-looking owner
(your own account, an org admin, the first user in a list) to keep moving
is guessing on the user's behalf. Prefer a `USER` owner by email when
unsure - a `GROUP` owner must match a Split Team name, not a Harness
user-group identifier.

The same rule applies if an owner turns out to be invalid rather than
missing (e.g. a 400 on create) - stop and ask for a real replacement, don't
silently substitute one.

## Step 6: "before"/trigger relationship vs a plain filter event

A base event can be scoped by another event in two independent ways, on
different fields, so an ambiguous request needs a pick before drafting:

| Concept | Meaning | Field |
|---------|---------|-------|
| Filter event (`HAS_DONE`) | Only count units that did this event at all | `filterEventType` with `filterAggregation: "RATE"` |
| Trigger event (`HAS_DONE_BEFORE`) | Only count units that did this event *before* the base event | `triggerEventType` (`{eventTypeId}` only - no aggregation/property filters) |
