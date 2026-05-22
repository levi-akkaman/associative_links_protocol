import argparse
import json
import sys
from pathlib import Path
from typing import Dict, Tuple

DB_PATH = Path("associations.json")


def load_db() -> Dict[int, Tuple[int, int]]:
    if not DB_PATH.exists():
        return {}
    try:
        with open(DB_PATH, encoding="utf-8") as f:
            raw = json.load(f)
        return {int(k): (int(v[0]), int(v[1])) for k, v in raw.items()}
    except Exception as e:
        print(f"Database read error: {e}", file=sys.stderr)
        sys.exit(1)


def save_db(db: Dict[int, Tuple[int, int]]) -> None:
    # convert to a JSON compatible format
    data = {str(k): [str(v[0]), str(v[1])] for k, v in sorted(db.items())}
    try:
        DB_PATH.parent.mkdir(parents=True, exist_ok=True)
        with open(DB_PATH, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    except Exception as e:
        print(f"Database write error: {e}", file=sys.stderr)
        sys.exit(1)


def get_next_id(db: Dict[int, Tuple[int, int]]) -> int:
    return max(db.keys()) + 1 if db else 1


def is_self_closed(cid: int, from_id: int, to_id: int) -> bool:
    # checking whether the circuit is self closing
    return from_id == to_id == cid


def is_referenced(db: Dict[int, Tuple[int, int]], cid: int) -> bool:
    # checks whether anyone is referencing this ID
    for id_, (f, t) in db.items():
        if id_ == cid:
            continue
        if f == cid or t == cid:
            return True
    return False


def validate_db(db: Dict[int, Tuple[int, int]], skip_cid: int = None) -> bool:
    # validation database
    all_ids = set(db.keys())
    for cid, (f, t) in db.items():
        if cid == skip_cid:
            continue
        if f not in all_ids or t not in all_ids:
            print(f"Integrity error: The link {cid} refers to a non-existent ID {f},{t}")
            return False
    return True


# ====================== COMANDS ======================

def cmd_init():
    # initializing a new database
    if DB_PATH.exists():
        print("The database has already been initialized")
        return
    save_db({})
    print(f"Database created: {DB_PATH.resolve()}")


def cmd_list():
    # display all links in log format
    db = load_db()
    if not db:
        print("Database is empty")
        return

    parts = []
    for cid in sorted(db.keys()):
        f, t = db[cid]
        mark = " [SELF]" if is_self_closed(cid, f, t) else ""
        parts.append(f"{cid}: {f},{t}{mark}")

    print(" | ".join(parts))
    print(f"\nTotal links: {len(db)}")


def cmd_create_self():
    # created self closing links
    db = load_db()
    nid = get_next_id(db)
    db[nid] = (nid, nid)
    save_db(db)
    print(f"Create self-closing links → {nid}: {nid},{nid}")


def cmd_connect(args):
    # create a link between existing objects
    db = load_db()
    f = args.from_id
    t = args.to_id

    if f not in db or t not in db:
        print("Error: from_id and to_id must exist in the database")
        print("Use this first 'create-self' or connect them to existing ones")
        sys.exit(1)

    nid = get_next_id(db)
    db[nid] = (f, t)
    save_db(db)

    print(f"Links created → {nid}: {f},{t}")


def cmd_delete(args):
    db = load_db()
    cid = args.id

    if cid not in db:
        print(f"Link {cid} not found")
        sys.exit(1)

    if is_referenced(db, cid):
        print(f"{cid} cannot be deleted it is referenced by other links")
        print("First, delete or modify all links that use this ID")
        sys.exit(1)

    del db[cid]
    save_db(db)
    print(f"Links delete {cid}")


def cmd_modify(args):
    db = load_db()
    cid = args.id

    if cid not in db:
        print(f"Link {cid} not found")
        sys.exit(1)

    new_f = args.from_id
    new_t = args.to_id

    if not (is_self_closed(cid, new_f, new_t) or (new_f in db and new_t in db)):
        print("Error: New from/to elements must be self-closing (id,id),")
        print("or refer to existing objects")
        sys.exit(1)

    old_f, old_t = db[cid]
    db[cid] = (new_f, new_t)
    save_db(db)

    print(f"Modify link {cid}: {old_f},{old_t} → {new_f},{new_t}")


def cmd_status():
    db = load_db()
    self_closed = sum(1 for cid, (f, t) in db.items() if is_self_closed(cid, f, t))
    total = len(db)
    print(f"Total links: {total}")
    print(f"   • self-closing: {self_closed}")
    print(f"   • dependents: {total - self_closed}")
    print(f"   database: {DB_PATH.resolve()}")


# ====================== CLI ======================

def main():
    parser = argparse.ArgumentParser(
        description="CLI for Associative Link",
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    subparsers = parser.add_subparsers(dest="command", required=True)

    # init
    subparsers.add_parser("init", help="Initialize a new database")

    # list
    subparsers.add_parser("list", help="Show all links")

    # create-self
    subparsers.add_parser("create-self", help="Create new self-closed links")

    # connect
    p_connect = subparsers.add_parser("connect", help="Create a relationship between two existing objects")
    p_connect.add_argument("from_id", type=int, help="Source object ID")
    p_connect.add_argument("to_id", type=int, help="Recipient Object ID")

    # delete
    p_delete = subparsers.add_parser("delete", help="delete link")
    p_delete.add_argument("id", type=int, help="ID  link for delete")

    # modify
    p_modify = subparsers.add_parser("modify", help="Change the “from/to” fields for an existing link")
    p_modify.add_argument("id", type=int, help="ID of the editable link")
    p_modify.add_argument("--from-id", type=int, required=True, dest="from_id", help="New from")
    p_modify.add_argument("--to-id", type=int, required=True, dest="to_id", help="New to")

    # status
    subparsers.add_parser("status", help="Status database")

    args = parser.parse_args()

    if args.command == "init":
        cmd_init()
    elif args.command == "list":
        cmd_list()
    elif args.command == "create-self":
        cmd_create_self()
    elif args.command == "connect":
        cmd_connect(args)
    elif args.command == "delete":
        cmd_delete(args)
    elif args.command == "modify":
        cmd_modify(args)
    elif args.command == "status":
        cmd_status()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()