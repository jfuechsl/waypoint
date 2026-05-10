Archive completed milestones. Only proceed after explicit user confirmation.

1. Read docs/roadmap.md. List all milestones with Status ✅ Complete.
   If none are complete, say so and stop.
2. Present the list to the user and ask which milestones to archive.
   (Can be all complete milestones or a subset.)
3. Wait for explicit confirmation before touching any files.
4. For each confirmed milestone:
   a. Move docs/milestones/ms-<slug>.md
      → docs/milestones/archive/ms-<slug>.md
   b. Remove the milestone's row from the active table in docs/roadmap.md.
   c. Append the row to docs/milestones/archive/index.md, adding the
      Completed date if not already recorded.
5. Summarize what was moved.
