---
name: run-decision-queue
description: >-
  Run a ruling session whenever you hold more than one open point for the
  owner to rule on: plan open questions, audit findings, review findings,
  deferred items. Stage the points in a task list, raise one at a time with
  context, options, and one recommendation, and record each ruling in its
  durable home before you raise the next. Use it instead of sending the points
  as one list. The project's documents own where rulings are recorded.
---

# Run a decision queue

When more than one point waits on the owner, do not send them as a list. A list asks for many
rulings at once, and the owner must open other sources to make each one. Walk the points as a
queue: one item, one ruling, one record, then the next item.

The project's documents own where a ruling lives: the issue tracker, the label for an open
decision, a design record, a plan file. Read them before you raise the first item. Where this
skill and the project disagree, the project wins.

## 1. Stage the queue

- Collect every open point before you raise the first one.
- Put the queue in a task list. Use the harness's task tool when it has one. Otherwise use a file
  in your scratchpad. Never put the queue in an untracked file in the project's working tree.
- Order by dependency first: raise a point before the points its ruling decides. Then follow the
  owner's stated priority.
- Group by topic, so related rulings sit together.

## 2. Verify before you raise

Just before you raise an item, check it against live state:

- The issue or PR is still open.
- The claim still holds. The file and line you cite still say what you say they do.
- An earlier ruling has not already settled it.

Drop a stale item, or rewrite it to match what is true now.

## 3. Raise one item

Raise one item per message. Show its position, as "item 3 of 11", so the owner can see the size
of the queue. Never list the content of the other items in advance.

Each item carries four parts:

1. **The decision**, in one line.
2. **The context** the owner needs to rule without opening anything else: what the point is, why
   it matters now, and the evidence as `file:line` references or links.
3. **Two to four options**, each with its consequence.
4. **Exactly one recommendation**, marked clearly, with its reason.

```
Item 3 of 11: <the decision, in one line>

<Context: what it is, why it matters now, evidence as file:line or links.>

A. <option>. Consequence: <...>
B. <option>. Consequence: <...>

Recommendation: A. <reason>
```

- Do not bundle other considerations before the question. Ask one thing per item.
- You may offer minor defaults as one "accept these defaults" item, only when each default is
  truly minor. List every default in that item.
- When the harness offers a structured question tool, it may carry the options, with the
  recommendation first. The context still goes in the message. Use the tool only for a crisp,
  stable fork between exclusive options. Put a nuanced or still-forming decision as prose: the
  trade-offs, the recommendation, and an invitation to steer. A choice modal on a question still
  being shaped pressures a premature answer.

## 4. Record the ruling

- Record the ruling in its durable home before you raise the next item: the item's issue or PR,
  or a new issue when it has none.
- Record the owner's rationale in the owner's words. Quote it. Do not paraphrase it.
- Never carry a ruling in chat across items.
- Update any plan or brief the ruling changes, and mark the item done in the task list.
- When the owner overrules your recommendation, record the ruling as given. Do not relitigate it.

## 5. Defer an item

When the owner defers an item, or it cannot be decided now, open a decision-tracking issue with
the project's label for open decisions (`needs-decision` or its equivalent). Put the options, the
context, and the consequences in the issue body in neutral terms. Post your recommendation as a
separate comment, so a reader can tell it apart from the neutral framing. Then raise the next
item.

## 6. The scope of a ruling

A ruling authorises exactly what it names. A generic "continue" or "carry on" does not approve
another open item or a pending plan. It does not lift a hold the owner placed.

## 7. Close the queue

When the queue is empty, send one short summary: each ruling in one line, with a link to where it
was recorded. Then continue with the work the rulings unblocked, within the scope they authorised.
