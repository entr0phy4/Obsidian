**Password**: dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
**Tools**: [[sort]], [[uniq]]

## Solution
The password for the [[Level 9]] is stored in the file **data.txt** and is the only line of text that occurs only once.

we can user [[uniq]] to display the only the line that occurs only once but need provide a sorted output with [[sort]]

```bash
cat data.txt | sort | uniq -u
```