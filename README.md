# remember-the-milk-statistics-contract

Shared data contract between the RTM Statistics producer and its consumers. It holds the
SQLite table schema as JSON. There is no application code here.

## Files

- `schema.json`: one entry per shared table, keyed by table name:
  - `columns`: column name to SQLite type (`PRAGMA table_info` type), sorted by name.
  - `direction`: `api_to_client`. The producer writes the table and consumers read it.
  - `description`: where the table lives and any format notes, such as the date format.

## Producer

- remember-the-milk-statistics (Python) writes `data/statistics.sqlite`.
  - `tools/export_contract.bat` regenerates `schema.json` from the real table definition.
  - `tests/unit/test_contract.py` fails when the table differs from `schema.json`.

## Consumers

- remember-the-milk-statistics-frontend (PHP) reads the table.
  - `tests/Unit/ContractTest.php`: every column the frontend reads must exist in `schema.json`
    with the same type. The frontend may read fewer columns than the contract lists.
  - `tests/Integration/SnapshotRepositoryTest.php` builds its test table from `schema.json`.

## Usage

Every project mounts this repository as a git submodule at `contract/`:

```bash
git submodule add https://github.com/BenjaminKobjolke/remember-the-milk-statistics-contract.git contract
git submodule update --remote contract   # pick up a contract change
```

Commit the pointer bump with the message `GIT: bump remember-the-milk-statistics-contract submodule`.
Production code never reads `contract/`. It is test input only, and the tests skip when it is
not checked out.

## Change order

- **Add a column:** add it in the producer and update `schema.json` in the same change. Then
  release the producer. Consumers may start reading it after that.
- **Remove a column:** stop reading it in every consumer first. Then remove it from
  `schema.json` and the producer.
- **Change a type:** treat it as a remove followed by an add.
- **Never rename a column.** A rename is an add followed by a remove, each in its safe order.
- Update this README in the same commit as every contract change.

## Validation

```bash
python -m json.tool schema.json
```
