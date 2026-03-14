# GST Code Example — In-Memory Task Manager

Gestalt v3.2 encoded version of CODE_EXAMPLE.md.
Compressed from ~4817 tokens to ~1768 tokens (63% reduction).

```
META☩code_syntax☩lang:python,lbrace_escape:☸,rbrace_escape:☥

DOC☩task_manager☩domain:python,type:in_memory_demo,safety:no_file_io_no_network☸

SEC☩enums☩level:1☸
CONCEPT☩Priority☩type:enum,domain:task☸LOW=1,MEDIUM=2,HIGH=3,CRITICAL=4☥
CONCEPT☩Status☩type:enum,domain:task☸pending,in_progress,blocked,complete,cancelled☥
CONCEPT☩Category☩type:enum,domain:task☸work,personal,learning,health,finance,other☥
☥

SEC☩models☩level:1☸

CLASS☩Tag☩access:public☸
stores tag name and color,id uuid generated,created_at timestamp☥
FUNC☩Tag.__init__☩params:name:str,color:Optional[str],return:None☸
strips and lowercases name,assigns uuid id,defaults color to grey☥
FUNC☩Tag.__eq__☩params:other,return:bool☸
equality by name only☥
FUNC☩Tag.__hash__☩params:none,return:int☸
hash by name☥

CLASS☩Comment☩access:public☸
stores author,body,timestamps,supports editing☥
FUNC☩Comment.__init__☩params:author:str,body:str,return:None☸
assigns uuid id,records created_at,edited_at None☥
FUNC☩Comment.edit☩params:new_body:str,return:None☸
updates body,sets edited_at to now☥

CLASS☩Task☩access:public☸
core data model,holds all task state,supports subtasks and dependencies☥
FUNC☩Task.__init__☩params:title:str,description:str,priority:Priority,category:Category,due_date:Optional[datetime],assignee:Optional[str],return:None☸
assigns uuid id,sets status to PENDING,initializes empty tags,comments,subtasks,dependencies lists☥
FUNC☩Task.add_tag☩params:tag:Tag,return:None☸
appends tag if not already present,updates updated_at☥
FUNC☩Task.remove_tag☩params:tag_name:str,return:bool☸
filters tags list by name,returns True if removed☥
FUNC☩Task.add_comment☩params:author:str,body:str,return:Comment☸
creates Comment,appends to comments,updates updated_at,returns comment☥
FUNC☩Task.add_subtask☩params:subtask:Task,return:None☸
appends subtask to subtasks list,updates updated_at☥
FUNC☩Task.add_dependency☩params:task_id:str,return:None☸
appends task_id if not already in dependencies☥
FUNC☩Task.set_status☩params:status:Status,return:None☸
sets status,updates updated_at,sets completed_at if COMPLETE☥
FUNC☩Task.set_estimate☩params:hours:float,return:None☸
sets estimated_hours,updates updated_at☥
FUNC☩Task.log_time☩params:hours:float,return:None☸
adds hours to actual_hours,updates updated_at☥
FUNC☩Task.is_overdue☩params:none,return:bool☸
returns True if due_date set and now exceeds it and status not COMPLETE or CANCELLED☥
FUNC☩Task.completion_percentage☩params:none,return:float☸
if no subtasks returns 100 if COMPLETE else 0,else calculates percentage of completed subtasks☥
☥

CLASS☩TaskStore☩access:public☸
in-memory dict store for tasks and tags,no persistence☥
FUNC☩TaskStore.__init__☩params:none,return:None☸
initializes empty _tasks and _tags dicts☥
FUNC☩TaskStore.add_task☩params:task:Task,return:Task☸
stores task by id,returns task☥
FUNC☩TaskStore.get_task☩params:task_id:str,return:Optional[Task]☸
returns task by id or None☥
FUNC☩TaskStore.remove_task☩params:task_id:str,return:bool☸
deletes task by id,returns True if found☥
FUNC☩TaskStore.get_or_create_tag☩params:name:str,color:Optional[str],return:Tag☸
returns existing tag by normalized name or creates new one☥
FUNC☩TaskStore.all_tasks☩params:none,return:List[Task]☸
returns all tasks as list☥
FUNC☩TaskStore.filter_by_status☩params:status:Status,return:List[Task]☸
filters tasks by status☥
FUNC☩TaskStore.filter_by_priority☩params:priority:Priority,return:List[Task]☸
filters tasks by priority☥
FUNC☩TaskStore.filter_by_category☩params:category:Category,return:List[Task]☸
filters tasks by category☥
FUNC☩TaskStore.filter_by_assignee☩params:assignee:str,return:List[Task]☸
filters tasks by assignee☥
FUNC☩TaskStore.filter_by_tag☩params:tag_name:str,return:List[Task]☸
filters tasks where any tag name matches☥
FUNC☩TaskStore.overdue_tasks☩params:none,return:List[Task]☸
returns all tasks where is_overdue is True☥
FUNC☩TaskStore.search☩params:query:str,return:List[Task]☸
case-insensitive search on title and description☥
FUNC☩TaskStore.summary☩params:none,return:Dict☸
returns total count,count by status,count by priority,overdue count☥
☥

CLASS☩TaskReporter☩access:public☸
generates formatted text reports from TaskStore☥
FUNC☩TaskReporter.task_detail☩params:task:Task,return:str☸
full formatted detail block including status,priority,dates,tags,dependencies,subtasks,comments☥
FUNC☩TaskReporter.summary_report☩params:none,return:str☸
formatted summary with totals by status and priority☥
FUNC☩TaskReporter.assignee_report☩params:assignee:str,return:str☸
lists all tasks assigned to given assignee with overdue flag☥
FUNC☩TaskReporter.blocked_report☩params:none,return:str☸
lists all blocked tasks with dependencies and last comment☥
☥

CLASS☩CommandProcessor☩access:public☸
processes string commands against TaskStore,dispatches to handler methods☥
CONCEPT☩supported_commands☩domain:interface☸
add,done,cancel,assign,tag,untag,comment,estimate,logtime,block,start,list,search,summary,detail,help☥
FUNC☩CommandProcessor.process☩params:command:str,args:List[str],return:str☸
dispatches to _cmd_{command} handler,returns error string if unknown command or exception☥
FUNC☩CommandProcessor._cmd_add☩params:args,return:str☸
creates Task from title args,adds to store,returns confirmation with short id☥
FUNC☩CommandProcessor._cmd_done☩params:args,return:str☸
sets task status to COMPLETE☥
FUNC☩CommandProcessor._cmd_cancel☩params:args,return:str☸
sets task status to CANCELLED☥
FUNC☩CommandProcessor._cmd_start☩params:args,return:str☸
sets task status to IN_PROGRESS☥
FUNC☩CommandProcessor._cmd_block☩params:args,return:str☸
sets task status to BLOCKED☥
FUNC☩CommandProcessor._cmd_assign☩params:args,return:str☸
sets task assignee☥
FUNC☩CommandProcessor._cmd_tag☩params:args,return:str☸
gets or creates tag,adds to task☥
FUNC☩CommandProcessor._cmd_untag☩params:args,return:str☸
removes tag from task by name☥
FUNC☩CommandProcessor._cmd_comment☩params:args,return:str☸
adds comment with author and body to task☥
FUNC☩CommandProcessor._cmd_estimate☩params:args,return:str☸
sets estimated hours on task☥
FUNC☩CommandProcessor._cmd_logtime☩params:args,return:str☸
logs hours to task actual_hours☥
FUNC☩CommandProcessor._cmd_list☩params:args,return:str☸
returns formatted list of all tasks with status and priority☥
FUNC☩CommandProcessor._cmd_search☩params:args,return:str☸
returns matching tasks for query string☥
FUNC☩CommandProcessor._cmd_detail☩params:args,return:str☸
returns full task detail report☥
FUNC☩CommandProcessor._cmd_summary☩params:args,return:str☸
returns summary report☥
FUNC☩CommandProcessor._cmd_help☩params:args,return:str☸
returns list of available commands☥
☥

SEC☩session_runner☩level:1☸
FUNC☩run_demo_session☩params:store:TaskStore,return:None☸
creates sample tags and 5 tasks across work,learning,finance categories,demonstrates status transitions,comments,dependencies,assignees☥
RELATES☩TaskStore☩depends_on
RELATES☩Task☩depends_on

PROTOCOL☩entry_point☩sequence:final☸
instantiates TaskStore,calls run_demo_session,prints summary☥
RELATES☩run_demo_session☩calls
RELATES☩TaskStore☩depends_on
☥

☥
```
