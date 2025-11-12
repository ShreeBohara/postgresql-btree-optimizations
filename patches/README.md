# Patch Files

This directory contains the patch file for applying B-tree optimizations to PostgreSQL 17.4.

## btree_optimizations.patch

This unified diff patch contains all modifications needed to add the two B-tree optimizations to PostgreSQL 17.4.

### Files Modified

- `src/include/access/nbtree.h` - External variable declarations
- `src/backend/access/nbtree/nbtutils.c` - Variable definitions
- `src/backend/access/nbtree/nbtsearch.c` - Core optimization logic
- `src/backend/utils/misc/guc_tables.c` - GUC system integration

### Applying the Patch

```bash
cd postgresql-17.4
git apply /path/to/patches/btree_optimizations.patch
```

Or with git:
```bash
cd postgresql-17.4
git init  # if not already a git repo
git add .
git commit -m "Initial commit"
git apply /path/to/patches/btree_optimizations.patch
```

### Verifying the Patch

After applying, verify the changes:
```bash
git diff --stat
```

You should see modifications to the four files listed above.

### Reverting the Patch

To remove the optimizations:
```bash
git checkout -- src/include/access/nbtree.h \
                 src/backend/access/nbtree/nbtutils.c \
                 src/backend/access/nbtree/nbtsearch.c \
                 src/backend/utils/misc/guc_tables.c
```

Or if using git:
```bash
git apply -R /path/to/patches/btree_optimizations.patch
```

