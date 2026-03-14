# Go Translation Example

**Source:** GST_CODE_EXAMPLE.md (Gestalt encoded version only — original Python not referenced)

**Purpose:** Demonstrates cross-language translation from Gestalt compressed source.

**Status:** Structurally verified by inspection. Compilation pending Go runtime validation.

**Token cost of translation source:** ~1768 tokens (Gestalt) vs ~4817 tokens (original Python)

---

```go
package main

import (
	"fmt"
	"strings"
	"time"

	"github.com/google/uuid"
)

// --- Enums ---

type Priority int

const (
	PriorityLow      Priority = 1
	PriorityMedium   Priority = 2
	PriorityHigh     Priority = 3
	PriorityCritical Priority = 4
)

func (p Priority) String() string {
	switch p {
	case PriorityLow:
		return "LOW"
	case PriorityMedium:
		return "MEDIUM"
	case PriorityHigh:
		return "HIGH"
	case PriorityCritical:
		return "CRITICAL"
	default:
		return "UNKNOWN"
	}
}

type Status string

const (
	StatusPending    Status = "pending"
	StatusInProgress Status = "in_progress"
	StatusBlocked    Status = "blocked"
	StatusComplete   Status = "complete"
	StatusCancelled  Status = "cancelled"
)

type Category string

const (
	CategoryWork     Category = "work"
	CategoryPersonal Category = "personal"
	CategoryLearning Category = "learning"
	CategoryHealth   Category = "health"
	CategoryFinance  Category = "finance"
	CategoryOther    Category = "other"
)

// --- Models ---

type Tag struct {
	ID        string
	Name      string
	Color     string
	CreatedAt time.Time
}

func NewTag(name string, color string) *Tag {
	if color == "" {
		color = "grey"
	}
	return &Tag{
		ID:        uuid.New().String(),
		Name:      strings.TrimSpace(strings.ToLower(name)),
		Color:     color,
		CreatedAt: time.Now(),
	}
}

type Comment struct {
	ID        string
	Author    string
	Body      string
	CreatedAt time.Time
	EditedAt  *time.Time
}

func NewComment(author, body string) *Comment {
	return &Comment{
		ID:        uuid.New().String(),
		Author:    author,
		Body:      body,
		CreatedAt: time.Now(),
	}
}

func (c *Comment) Edit(newBody string) {
	c.Body = newBody
	now := time.Now()
	c.EditedAt = &now
}

type Task struct {
	ID              string
	Title           string
	Description     string
	Priority        Priority
	Category        Category
	Status          Status
	DueDate         *time.Time
	Assignee        string
	CreatedAt       time.Time
	UpdatedAt       time.Time
	CompletedAt     *time.Time
	Tags            []*Tag
	Comments        []*Comment
	Subtasks        []*Task
	Dependencies    []string
	EstimatedHours  *float64
	ActualHours     float64
}

func NewTask(title, description string, priority Priority, category Category) *Task {
	now := time.Now()
	return &Task{
		ID:          uuid.New().String(),
		Title:       title,
		Description: description,
		Priority:    priority,
		Category:    category,
		Status:      StatusPending,
		CreatedAt:   now,
		UpdatedAt:   now,
	}
}

func (t *Task) AddTag(tag *Tag) {
	for _, existing := range t.Tags {
		if existing.Name == tag.Name {
			return
		}
	}
	t.Tags = append(t.Tags, tag)
	t.UpdatedAt = time.Now()
}

func (t *Task) RemoveTag(tagName string) bool {
	name := strings.ToLower(tagName)
	original := len(t.Tags)
	filtered := t.Tags[:0]
	for _, tag := range t.Tags {
		if tag.Name != name {
			filtered = append(filtered, tag)
		}
	}
	t.Tags = filtered
	if len(t.Tags) < original {
		t.UpdatedAt = time.Now()
		return true
	}
	return false
}

func (t *Task) AddComment(author, body string) *Comment {
	comment := NewComment(author, body)
	t.Comments = append(t.Comments, comment)
	t.UpdatedAt = time.Now()
	return comment
}

func (t *Task) AddSubtask(subtask *Task) {
	t.Subtasks = append(t.Subtasks, subtask)
	t.UpdatedAt = time.Now()
}

func (t *Task) AddDependency(taskID string) {
	for _, dep := range t.Dependencies {
		if dep == taskID {
			return
		}
	}
	t.Dependencies = append(t.Dependencies, taskID)
	t.UpdatedAt = time.Now()
}

func (t *Task) SetStatus(status Status) {
	t.Status = status
	t.UpdatedAt = time.Now()
	if status == StatusComplete {
		now := time.Now()
		t.CompletedAt = &now
	}
}

func (t *Task) SetEstimate(hours float64) {
	t.EstimatedHours = &hours
	t.UpdatedAt = time.Now()
}

func (t *Task) LogTime(hours float64) {
	t.ActualHours += hours
	t.UpdatedAt = time.Now()
}

func (t *Task) IsOverdue() bool {
	if t.DueDate != nil && t.Status != StatusComplete && t.Status != StatusCancelled {
		return time.Now().After(*t.DueDate)
	}
	return false
}

func (t *Task) CompletionPercentage() float64 {
	if len(t.Subtasks) == 0 {
		if t.Status == StatusComplete {
			return 100.0
		}
		return 0.0
	}
	completed := 0
	for _, sub := range t.Subtasks {
		if sub.Status == StatusComplete {
			completed++
		}
	}
	return float64(completed) / float64(len(t.Subtasks)) * 100
}

// --- Task Store ---

type TaskStore struct {
	tasks map[string]*Task
	tags  map[string]*Tag
}

func NewTaskStore() *TaskStore {
	return &TaskStore{
		tasks: make(map[string]*Task),
		tags:  make(map[string]*Tag),
	}
}

func (s *TaskStore) AddTask(task *Task) *Task {
	s.tasks[task.ID] = task
	return task
}

func (s *TaskStore) GetTask(taskID string) *Task {
	return s.tasks[taskID]
}

func (s *TaskStore) RemoveTask(taskID string) bool {
	if _, ok := s.tasks[taskID]; ok {
		delete(s.tasks, taskID)
		return true
	}
	return false
}

func (s *TaskStore) GetOrCreateTag(name, color string) *Tag {
	key := strings.TrimSpace(strings.ToLower(name))
	if tag, ok := s.tags[key]; ok {
		return tag
	}
	tag := NewTag(name, color)
	s.tags[key] = tag
	return tag
}

func (s *TaskStore) AllTasks() []*Task {
	tasks := make([]*Task, 0, len(s.tasks))
	for _, t := range s.tasks {
		tasks = append(tasks, t)
	}
	return tasks
}

func (s *TaskStore) FilterByStatus(status Status) []*Task {
	var result []*Task
	for _, t := range s.tasks {
		if t.Status == status {
			result = append(result, t)
		}
	}
	return result
}

func (s *TaskStore) FilterByAssignee(assignee string) []*Task {
	var result []*Task
	for _, t := range s.tasks {
		if t.Assignee == assignee {
			result = append(result, t)
		}
	}
	return result
}

func (s *TaskStore) Search(query string) []*Task {
	q := strings.ToLower(query)
	var result []*Task
	for _, t := range s.tasks {
		if strings.Contains(strings.ToLower(t.Title), q) ||
			strings.Contains(strings.ToLower(t.Description), q) {
			result = append(result, t)
		}
	}
	return result
}

func (s *TaskStore) Summary() map[string]interface{} {
	tasks := s.AllTasks()
	byStatus := map[string]int{}
	byPriority := map[string]int{}
	overdue := 0
	for _, t := range tasks {
		byStatus[string(t.Status)]++
		byPriority[t.Priority.String()]++
		if t.IsOverdue() {
			overdue++
		}
	}
	return map[string]interface{}{
		"total":       len(tasks),
		"by_status":   byStatus,
		"by_priority": byPriority,
		"overdue":     overdue,
	}
}

// --- Demo Session ---

func runDemoSession(store *TaskStore) {
	urgent := store.GetOrCreateTag("urgent", "red")
	review := store.GetOrCreateTag("needs-review", "yellow")
	blockedTag := store.GetOrCreateTag("blocked", "orange")

	t1 := NewTask(
		"Finalize API authentication layer",
		"Complete OAuth2 implementation and write integration tests",
		PriorityCritical,
		CategoryWork,
	)
	t1.Assignee = "alice"
	t1.SetEstimate(8.0)
	t1.AddTag(urgent)
	t1.AddComment("alice", "Blocked on security review approval")
	t1.SetStatus(StatusBlocked)
	t1.AddTag(blockedTag)
	store.AddTask(t1)

	t2 := NewTask(
		"Write unit tests for data pipeline",
		"Cover edge cases in transformation and validation steps",
		PriorityHigh,
		CategoryWork,
	)
	t2.Assignee = "bob"
	t2.SetEstimate(4.0)
	t2.LogTime(1.5)
	t2.SetStatus(StatusInProgress)
	t2.AddDependency(t1.ID)
	store.AddTask(t2)

	t3 := NewTask(
		"Update onboarding documentation",
		"Reflect recent changes to setup process",
		PriorityMedium,
		CategoryWork,
	)
	t3.Assignee = "carol"
	t3.AddTag(review)
	t3.AddComment("carol", "First draft complete, needs technical review")
	store.AddTask(t3)

	t4 := NewTask(
		"Read distributed systems paper",
		"Review the Raft consensus algorithm paper",
		PriorityLow,
		CategoryLearning,
	)
	store.AddTask(t4)

	t5 := NewTask(
		"Quarterly budget review",
		"Review Q3 actuals and prepare Q4 forecast",
		PriorityHigh,
		CategoryFinance,
	)
	t5.SetEstimate(2.0)
	store.AddTask(t5)

	summary := store.Summary()
	fmt.Printf("Session summary: %v\n", summary)

	blocked := store.FilterByStatus(StatusBlocked)
	fmt.Printf("Blocked tasks: ")
	for _, t := range blocked {
		fmt.Printf("[%s priority=%s] ", t.Title, t.Priority)
	}
	fmt.Println()

	aliceTasks := store.FilterByAssignee("alice")
	fmt.Printf("Alice's tasks: ")
	for _, t := range aliceTasks {
		fmt.Printf("[%s status=%s] ", t.Title, t.Status)
	}
	fmt.Println()

	pipelineResults := store.Search("pipeline")
	fmt.Printf("Search 'pipeline': ")
	for _, t := range pipelineResults {
		fmt.Printf("[%s status=%s] ", t.Title, t.Status)
	}
	fmt.Println()
}

func main() {
	store := NewTaskStore()
	runDemoSession(store)
}
```
