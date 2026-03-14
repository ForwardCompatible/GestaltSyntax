# Code Example — In-Memory Task Manager

A 566-line Python task manager. No file I/O, no network calls, no system modifications.
All data exists only in memory for the duration of the session.

This file contains intentional errors for debugging comparison testing.

```python
"""
In-memory task manager — demonstration script for Gestalt compression testing.
No file I/O, no network calls, no system modifications.
All data exists only in memory for the duration of the session.
"""

from typing import Optional, List, Dict
from datetime import datetime
from enum import Enum
import uuid


# --- Enums ---

class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4


class Status(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    BLOCKED = "blocked"
    COMPLETE = "complete"
    CANCELLED = "cancelled"


class Category(Enum):
    WORK = "work"
    PERSONAL = "personal"
    LEARNING = "learning"
    HEALTH = "health"
    FINANCE = "finance"
    OTHER = "other"


# --- Data Models ---

class Tag:
    def __init__(self, name: str, color: Optional[str] = None):
        self.id = str(uuid.uuid4())
        self.name = name.strip().lower()
        self.color = color or "grey"
        self.created_at = datetime.now()

    def __repr__(self):
        return f"Tag(name={self.name!r}, color={self.color!r})"

    def __eq__(self, other):
        if isinstance(other, Tag):
            return self.name == other.name
        return False

    def __hash__(self):
        return hash(self.name)


class Comment:
    def __init__(self, author: str, body: str):
        self.id = str(uuid.uuid4())
        self.author = author
        self.body = body
        self.created_at = datetime.now()
        self.edited_at: Optional[datetime] = None

    def edit(self, new_body: str) -> None:
        self.body = new_body
        self.edited_at = datetime.now()

    def __repr__(self):
        return f"Comment(author={self.author!r}, body={self.body[:30]!r})"


class Task:
    def __init__(
        self,
        title: str,
        description: str = "",
        priority: Priority = Priority.MEDIUM,
        category: Category = Category.OTHER,
        due_date: Optional[datetime] = None,
        assignee: Optional[str] = None,
    ):
        self.id = str(uuid.uuid4())
        self.title = title
        self.description = description
        self.priority = priority
        self.category = category
        self.status = Status.PENDING
        self.due_date = due_date
        self.assignee = assignee
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
        self.completed_at: Optional[datetime] = None
        self.tags: List[Tag] = []
        self.comments: List[Comment] = []
        self.subtasks: List["Task"] = []
        self.dependencies: List[str] = []
        self.estimated_hours: Optional[float] = None
        self.actual_hours: Optional[float] = None

    def add_tag(self, tag: Tag) -> None:
        if tag not in self.tags:
            self.tags.append(tag)
            self.updated_at = datetime.now()

    def remove_tag(self, tag_name: str) -> bool:
        original_len = len(self.tags)
        self.tags = [t for t in self.tags if t.name != tag_name.lower()]
        if len(self.tags) < original_len:
            self.updated_at = datetime.now()
            return True
        return False

    def add_comment(self, author: str, body: str) -> Comment:
        comment = Comment(author=author, body=body)
        self.comments.append(comment)
        self.updated_at = datetime.now()
        return comment

    def add_subtask(self, subtask: "Task") -> None:
        self.subtasks.append(subtask)
        self.updated_at = datetime.now()

    def add_dependency(self, task_id: str) -> None:
        if task_id not in self.dependencies:
            self.dependencies.append(task_id)
            self.updated_at = datetime.now()

    def set_status(self, status: Status) -> None:
        self.status = status
        self.updated_at = datetime.now()
        if status == Status.COMPLETE:
            self.completed_at = datetime.now()

    def set_estimate(self, hours: float) -> None:
        self.estimated_hours = hours
        self.updated_at = datetime.now()

    def log_time(self, hours: float) -> None:
        self.actual_hours = (self.actual_hours or 0) + hours
        self.updated_at = datetime.now()

    def is_overdue(self) -> bool:
        if self.due_date and self.status not in (Status.COMPLETE, Status.CANCELLED):
            return datetime.now() > self.due_date
        return False

    def completion_percentage(self) -> float:
        if not self.subtasks:
            return 100.0 if self.status == Status.COMPLETE else 0.0
        completed = sum(1 for t in self.subtasks if t.status == Status.COMPLETE)
        return round((completed / len(self.subtasks)) * 100, 1)

    def __repr__(self):
        return f"Task(title={self.title!r}, status={self.status.value}, priority={self.priority.name})"


# --- Task Store ---

class TaskStore:
    def __init__(self):
        self._tasks: Dict[str, Task] = {}
        self._tags: Dict[str, Tag] = {}

    def add_task(self, task: Task) -> Task:
        self._tasks[task.id] = task
        return task

    def get_task(self, task_id: str) -> Optional[Task]:
        return self._tasks.get(task_id)

    def remove_task(self, task_id: str) -> bool:
        if task_id in self._tasks:
            del self._tasks[task_id]
            return True
        return False

    def get_or_create_tag(self, name: str, color: Optional[str] = None) -> Tag:
        key = name.strip().lower()
        if key not in self._tags:
            self._tags[key] = Tag(name=key, color=color or "grey")
        return self._tags[key]

    def all_tasks(self) -> List[Task]:
        return list(self._tasks.values())

    def filter_by_status(self, status: Status) -> List[Task]:
        return [t for t in self._tasks.values() if t.status == status]

    def filter_by_priority(self, priority: Priority) -> List[Task]:
        return [t for t in self._tasks.values() if t.priority == priority]

    def filter_by_category(self, category: Category) -> List[Task]:
        return [t for t in self._tasks.values() if t.category == category]

    def filter_by_assignee(self, assignee: str) -> List[Task]:
        return [t for t in self._tasks.values() if t.assignee == assignee]

    def filter_by_tag(self, tag_name: str) -> List[Task]:
        key = tag_name.strip().lower()
        return [t for t in self._tasks.values() if any(tag.name == key for tag in t.tags)]

    def overdue_tasks(self) -> List[Task]:
        return [t for t in self._tasks.values() if t.is_overdue()]

    def search(self, query: str) -> List[Task]:
        query = query.lower()
        return [
            t for t in self._tasks.values()
            if query in t.title.lower() or query in t.description.lower()
        ]

    def summary(self) -> Dict:
        tasks = self.all_tasks()
        return {
            "total": len(tasks),
            "by_status": {s.value: sum(1 for t in tasks if t.status == s) for s in Status},
            "by_priority": {p.name: sum(1 for t in tasks if t.priority == p) for p in Priority},
            "overdue": len(self.overdue_tasks()),
        }


# --- Session Runner ---

def run_demo_session(store: TaskStore) -> None:
    urgent = store.get_or_create_tag("urgent", color="red")
    review = store.get_or_create_tag("needs-review", color="yellow")
    blocked_tag = store.get_or_create_tag("blocked", color="orange")

    t1 = Task(
        title="Finalize API authentication layer",
        description="Complete OAuth2 implementation and write integration tests",
        priority=Priority.CRITICAL,
        category=Category.WORK,
        assignee="alice",
    )
    t1.set_estimate(8.0)
    t1.add_tag(urgent)
    t1.add_comment("alice", "Blocked on security review approval")
    t1.set_status(Status.BLOCKED)
    t1.add_tag(blocked_tag)
    store.add_task(t1)

    t2 = Task(
        title="Write unit tests for data pipeline",
        description="Cover edge cases in transformation and validation steps",
        priority=Priority.HIGH,
        category=Category.WORK,
        assignee="bob",
    )
    t2.set_estimate(4.0)
    t2.log_time(1.5)
    t2.set_status(Status.IN_PROGRESS)
    t2.add_dependency(t1.id)
    store.add_task(t2)

    t3 = Task(
        title="Update onboarding documentation",
        description="Reflect recent changes to setup process",
        priority=Priority.MEDIUM,
        category=Category.WORK,
        assignee="carol",
    )
    t3.add_tag(review)
    t3.add_comment("carol", "First draft complete, needs technical review")
    store.add_task(t3)

    t4 = Task(
        title="Read distributed systems paper",
        description="Review the Raft consensus algorithm paper",
        priority=Priority.LOW,
        category=Category.LEARNING,
    )
    store.add_task(t4)

    t5 = Task(
        title="Quarterly budget review",
        description="Review Q3 actuals and prepare Q4 forecast",
        priority=Priority.HIGH,
        category=Category.FINANCE,
    )
    t5.set_estimate(2.0)
    store.add_task(t5)

    print("Session summary:", store.summary())
    print("Blocked tasks:", store.filter_by_status(Status.BLOCKED))
    print("Alice's tasks:", store.filter_by_assignee("alice"))
    print("Search 'pipeline':", store.search("pipeline"))


def main() -> None:
    store = TaskStore()
    run_demo_session(store)


if __name__ == "__main__":
    main()


# --- Reporter ---

class TaskReporter:
    """Generates formatted text reports from a TaskStore."""

    def __init__(self, store: TaskStore):
        self.store = store

    def _divider(self, width: int = 60) -> str:
        return "-" * width

    def _header(self, title: str, width: int = 60) -> str:
        padding = (width - len(title) - 2) // 2
        return f"{'=' * padding} {title} {'=' * padding}"

    def task_detail(self, task: Task) -> str:
        lines = [
            self._header(task.title),
            f"ID:          {task.id}",
            f"Status:      {task.status.value}",
            f"Priority:    {task.priority.name}",
            f"Category:    {task.category.value}",
            f"Assignee:    {task.assignee or 'unassigned'}",
            f"Created:     {task.created_at.strftime('%Y-%m-%d %H:%M')}",
            f"Updated:     {task.updated_at.strftime('%Y-%m-%d %H:%M')}",
        ]
        if task.due_date:
            overdue = " [OVERDUE]" if task.is_overdue() else ""
            lines.append(f"Due:         {task.due_date.strftime('%Y-%m-%d')}{overdue}")
        if task.estimated_hours:
            lines.append(f"Estimated:   {task.estimated_hours}h")
        if task.actual_hours:
            lines.append(f"Logged:      {task.actual_hours}h")
        if task.description:
            lines += [self._divider(), "Description:", f"  {task.description}"]
        if task.tags:
            lines += [self._divider(), "Tags:", f"  {', '.join(t.name for t in task.tags)}"]
        if task.dependencies:
            lines += [self._divider(), "Dependencies:", f"  {', '.join(task.dependencies)}"]
        if task.subtasks:
            lines += [self._divider(), f"Subtasks ({task.completion_percentage()}% complete):"]
            for sub in task.subtasks:
                lines.append(f"  [{sub.status.value}] {sub.title}")
        if task.comments:
            lines += [self._divider(), "Comments:"]
            for c in task.comments:
                lines.append(f"  {c.author} @ {c.created_at.strftime('%Y-%m-%d %H:%M')}: {c.body}")
        lines.append(self._divider())
        return "\n".join(lines)

    def summary_report(self) -> str:
        summary = self.store.summary()
        lines = [
            self._header("TASK SUMMARY REPORT"),
            f"Total tasks: {summary['total']}",
            f"Overdue:     {summary['overdue']}",
            self._divider(),
            "By Status:",
        ]
        for status, count in summary["by_status"].items():
            lines.append(f"  {status:<15} {count}")
        lines += [self._divider(), "By Priority:"]
        for priority, count in summary["by_priority"].items():
            lines.append(f"  {priority:<15} {count}")
        lines.append(self._divider())
        return "\n".join(lines)

    def assignee_report(self, assignee: str) -> str:
        tasks = self.store.filter_by_assignee(assignee)
        lines = [
            self._header(f"TASKS: {assignee.upper()}"),
            f"Total assigned: {len(tasks)}",
            self._divider(),
        ]
        for task in tasks:
            status_flag = "[!]" if task.is_overdue() else "   "
            lines.append(f"{status_flag} [{task.priority.name:<8}] {task.status.value:<12} {task.title}")
        lines.append(self._divider())
        return "\n".join(lines)

    def blocked_report(self) -> str:
        tasks = self.store.filter_by_status(Status.BLOCKED)
        lines = [
            self._header("BLOCKED TASKS"),
            f"Total blocked: {len(tasks)}",
            self._divider(),
        ]
        for task in tasks:
            lines.append(f"  {task.title}")
            if task.dependencies:
                lines.append(f"    Depends on: {', '.join(task.dependencies)}")
            if task.comments:
                last = task.comments[-1]
                lines.append(f"    Last comment: {last.author}: {last.body}")
        lines.append(self._divider())
        return "\n".join(lines)


# --- Command Processor ---

class CommandProcessor:
    """Processes simple string commands against a TaskStore."""

    COMMANDS = [
        "add", "done", "cancel", "assign", "tag", "untag",
        "comment", "estimate", "logtime", "block", "start",
        "list", "search", "summary", "detail", "help"
    ]

    def __init__(self, store: TaskStore):
        self.store = store
        self.reporter = TaskReporter(store)

    def process(self, command: str, args: List[str]) -> str:
        handler = getattr(self, f"_cmd_{command}", None)
        if handler is None:
            return f"Unknown command: {command!r}. Type 'help' for available commands."
        try:
            return handler(args)
        except Exception as e:
            return f"Error executing {command!r}: {e}"

    def _cmd_help(self, args: List[str]) -> str:
        return "Available commands: " + ", ".join(self.COMMANDS)

    def _cmd_summary(self, args: List[str]) -> str:
        return self.reporter.summary_report()

    def _cmd_list(self, args: List[str]) -> str:
        tasks = self.store.all_tasks()
        if not tasks:
            return "No tasks found."
        lines = []
        for t in tasks:
            lines.append(f"[{t.status.value:<12}] [{t.priority.name:<8}] {t.title}")
        return "\n".join(lines)

    def _cmd_search(self, args: List[str]) -> str:
        if not args:
            return "Usage: search <query>"
        query = " ".join(args)
        results = self.store.search(query)
        if not results:
            return f"No tasks matching {query!r}"
        return "\n".join(f"  {t.id[:8]} {t.title}" for t in results)

    def _cmd_detail(self, args: List[str]) -> str:
        if not args:
            return "Usage: detail <task_id>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        return self.reporter.task_detail(task)

    def _cmd_done(self, args: List[str]) -> str:
        if not args:
            return "Usage: done <task_id>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        task.set_status(Status.COMPLETE)
        return f"Marked complete: {task.title}"

    def _cmd_cancel(self, args: List[str]) -> str:
        if not args:
            return "Usage: cancel <task_id>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        task.set_status(Status.CANCELLED)
        return f"Cancelled: {task.title}"

    def _cmd_start(self, args: List[str]) -> str:
        if not args:
            return "Usage: start <task_id>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        task.set_status(Status.IN_PROGRESS)
        return f"Started: {task.title}"

    def _cmd_block(self, args: List[str]) -> str:
        if not args:
            return "Usage: block <task_id>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        task.set_status(Status.BLOCKED)
        return f"Marked blocked: {task.title}"

    def _cmd_assign(self, args: List[str]) -> str:
        if len(args) < 2:
            return "Usage: assign <task_id> <assignee>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        task.assignee = args[1]
        task.updated_at = datetime.now()
        return f"Assigned {task.title!r} to {args[1]}"

    def _cmd_tag(self, args: List[str]) -> str:
        if len(args) < 2:
            return "Usage: tag <task_id> <tag_name>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        tag = self.store.get_or_create_tag(args[1])
        task.add_tag(tag)
        return f"Tagged {task.title!r} with {args[1]!r}"

    def _cmd_untag(self, args: List[str]) -> str:
        if len(args) < 2:
            return "Usage: untag <task_id> <tag_name>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        removed = task.remove_tag(args[1])
        if removed:
            return f"Removed tag {args[1]!r} from {task.title!r}"
        return f"Tag {args[1]!r} not found on {task.title!r}"

    def _cmd_comment(self, args: List[str]) -> str:
        if len(args) < 3:
            return "Usage: comment <task_id> <author> <body>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        body = " ".join(args[2:])
        task.add_comment(args[1], body)
        return f"Comment added to {task.title!r}"

    def _cmd_estimate(self, args: List[str]) -> str:
        if len(args) < 2:
            return "Usage: estimate <task_id> <hours>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        try:
            hours = float(args[1])
        except ValueError:
            return f"Invalid hours value: {args[1]}"
        task.set_estimate(hours)
        return f"Set estimate for {task.title!r}: {hours}h"

    def _cmd_logtime(self, args: List[str]) -> str:
        if len(args) < 2:
            return "Usage: logtime <task_id> <hours>"
        task = self.store.get_task(args[0])
        if not task:
            return f"Task not found: {args[0]}"
        try:
            hours = float(args[1])
        except ValueError:
            return f"Invalid hours value: {args[1]}"
        task.log_time(hours)
        return f"Logged {hours}h on {task.title!r} (total: {task.actual_hours}h)"

    def _cmd_add(self, args: List[str]) -> str:
        if not args:
            return "Usage: add <title>"
        title = " ".join(args)
        task = Task(title=title)
        self.store.add_task(task)
        return f"Created task {task.id[:8]}: {task.title!r}"
```
