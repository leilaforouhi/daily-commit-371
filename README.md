
import json
from pathlib import Path

DATABASE = Path("file_snapshot.json")

def load_snapshot():
    if DATABASE.exists():
        return json.loads(
            DATABASE.read_text(encoding="utf-8")
        )
    return {}

def save_snapshot(snapshot):
    DATABASE.write_text(
        json.dumps(snapshot, indent=4),
        encoding="utf-8"
    )

def scan_directory(directory):
    root = Path(directory)
    snapshot = {}

    if not root.exists() or not root.is_dir():
        print("Invalid directory.")
        return None

    for file in root.rglob("*"):
        if file.is_file():
            relative = str(file.relative_to(root))

            try:
                stat = file.stat()

                snapshot[relative] = {
                    "size": stat.st_size,
                    "modified": stat.st_mtime
                }
            except OSError:
                continue

    return snapshot

def compare_snapshots(old, new):
    old_files = set(old)
    new_files = set(new)

    added = new_files - old_files
    deleted = old_files - new_files
    modified = []

    for filename in old_files & new_files:
        if old[filename] != new[filename]:
            modified.append(filename)

    print("\n===== FILE CHANGE REPORT =====")

    for filename in sorted(added):
        print(f"[ADDED] {filename}")

    for filename in sorted(modified):
        print(f"[MODIFIED] {filename}")

    for filename in sorted(deleted):
        print(f"[DELETED] {filename}")

    if not added and not modified and not deleted:
        print("No changes detected.")

    return added, modified, deleted

def main():
    directory = input("Directory to monitor: ").strip()

    current = scan_directory(directory)

    if current is None:
        return

    previous = load_snapshot()

    if not previous:
        print("Initial snapshot created.")
    else:
        compare_snapshots(previous, current)

    save_snapshot(current)
    print("\nSnapshot saved successfully!")

if __name__ == "__main__":
    main()
