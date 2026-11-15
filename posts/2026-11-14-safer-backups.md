# Safer backups

*14 November 2026.*

C: is the netBook's working disk, and it lives in RAM. A flat battery, or loading a new image off the card, and it's gone. PsiBoot offers to back C: up to the card before switching images.

PsiBoot now keeps up to three dated backups. Each one is written to a temporary folder first, and every file is read back and checked before it gets its proper name. The oldest is only removed once the new one is complete. If C: suddenly looks much smaller than the last backup, it warns you before going ahead. PsiTerm's keys and saved passwords stay off the card.

To restore, pick any of the backups from the list. Restoring copies it over C: and deletes nothing.
