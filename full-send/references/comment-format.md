## Comment format

Every comment you post on the user's behalf starts with this GitHub note callout.
Apply it to top-level PR comments, inline review comments, replies to bots or
reviewers, and written explanations used before you resolve or dismiss feedback:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**

Actual comment starts here.
```

- Use the verified public name of the model that wrote the comment, and the
  user's name or GitHub login. If the harness does not expose a model name, use
  the harness name. Do not guess.
- Keep the attribution inside the note and the reply outside it. Do not add a
  horizontal rule, heading, signature footer, or a second attribution.
- Do not add it to PR descriptions, commit messages, or text the user asked to
  post verbatim.
- Before posting, check the whole comment, including quoted text, for secrets and
  private details.
- Write the comment directly. Lead with the conclusion. Cite the code, behavior,
  check, or reproduction. Distinguish verified facts from judgment. Avoid filler.
- Preview the Markdown before posting. After posting, verify that GitHub returned
  the comment or review URL. Resolve a thread only after its reply posted.
