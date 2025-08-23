Welcome to the Linux Lab!

Login: `vagrant ssh`
User: `student` / Pass: `student123`

Read tasks in `~/tasks/taskX.txt`.
Solve tasks, then verify with:

  check_task.sh <task_id>

Examples:
  check_task.sh 1.1
  check_task.sh 4.2

Notes:
- Many checks require root; use `sudo -i`.
- For Task 7.1, configure static IP on the second adapter (likely `eth1`).
- Extra disk for storage tasks is attached as `/dev/sdb`.
