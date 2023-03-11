## Ansible Inventory

### Hostgroups

There are 2 types of groups

1. Default Groups
   1. All → this group will contain all hosts
   2. Ungrouped → all hosts that don’t have any other group apart from ‘all’.
2. Multiple Groups
   1. What (function) → based on app/stack/microservice (eg. dbserver, webserver, appserver)
   2. Where (location) → where the host is located (eg. datacenter or region like mumbai, east, west, local)
   3. When (environment) → development/production/staging etc.

## Patterns in Ansible

How to reach various hostgroups or hosts using pattern calls in Ansible

```toml
# Example inventory file

[group1]
host1
host2

[group2]
host1
host3

group1[0] # Reaches host1
group2[0] # Reaches host1
group1:&group2 # Reaches host1
group1:group2 # Reaches all hosts
group1:!group2 # Reaches host2 (Reach a host that is in group1 but not in group2)
group2:!group1 # Reaches host3 (Reach a host that is in group2 but not in group1)

# Shortening multiple similar URLs/Hosts
example1.com
example2.com -> example[1:3].com
example3.com

192.168.0.1
192.168.0.2 -> 192.168.0.[1:3]
192.168.0.3
```

## Ansible Ad-Hoc Commands:- for rarely executed tasks

- Execute the task on one node or multiple nodes

```bash
# Normal syntax to execute Ansible Ad-Hoc commands
ansible <host/host_group> -m <module_name> -a <args> -u <user_name> --become

# -m calls the module, -a passes an argument, -u use a particular user to perform the action, --become performs the action as the root user.
```

## Modules

https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html

- Modules are called upon using the `-m` syntax in an ad-hoc command
- Some examples of modules include `ping`, `uptime`, `date`, `debug`, etc.

### Ping Module

Is used to ping the nodes controlled by Ansible to verify connectivity.

```bash
# Example usage of the ping module
ansible <host/group> -m ping

# Ping a hostgroup using a particular user
ansible <host/group> -m ping -u <user>
```

### Debug Module

Save the output of the command into a register variable

```bash
# Print variables to console using the debug module
ansible <host/group> -m ping -m debug -a "msg=<yourmessagehere>"
```

### Setup Module

```bash
# Using the ansible setup module
ansible <host/group> -m setup
```

### Shell Module

Execute commands inside a node using the shell module.

```bash
# Executing a 'tree' commands inside a node using shell module
ansible <host/group> -m shell -a "tree --version"
ansible <host/group> -m shell -a "cat /home/downloads/download/txt"
```

### APT module

Use the apt module to install software on nodes

```bash
# Using the apt module to install software on nodes
ansible <host/group> -m apt -a "name=<package> state=present"

# Using the apt module to remove software on nodes
ansible <host/group> -m apt -a "name=<package> state=absent"
```

### File Module

Create files on nodes

```bash
# Creating files on nodes using the file module
ansible <host/group> -m file -a "dest=/root/sample.txt state=touch mode=600 owner=root group=root" --become
```

### Copy Module

Copy files or file content inside nodes

```bash
# Copying a file from src to dest inside a node
ansible <host/group> -m copy -a "src=/tmp/src_file dest=/home/usr/"

# Copying content of a file to another file
ansible <host/group> -m copy -a "content='Hello from Ansible CS' dest=/home/usr/src_file"
```

### Ansible Docs

Access Ansible documentation through the command line

```bash
# Accessing ansible docs from CLI
ansible-doc -l
ansible-doc -h

# Ansible documentation for a module
ansible-doc <module>
```

### Parallel Execution

```bash
ansible <host/group> -m shell -a <hostname> -f 1
ansible <host/group> -m shell -a <hostname> -f 2
```

## Ansible Loops

You can use 3 keywords to loop in Ansible. The variable name to use in loop is ‘item’.

### loop

This keyword loops through a list of items.

- only input allowed is a list

- requires implicit single-level flattening

  Eg. [1, [2, 3], 4] is not directly loop-able. It needs to be flattened.

  `loop: “{{ [1, [2, 3], 4] | flatten }}”`

```yaml
---
- hosts: all
  tasks:
    - name: create files
      file:
        path: "~/{{ item }}"
        state: touch
      loop:
        - test1.txt
        - test2.txt
```

### with_<lookup>

This keyword relies on lookup plugins (eg. items)

```yaml
---
- hosts: localhost
  tasks:
  - name: loop
      debug:
        msg: "{{ item }}"
			loop:
				- one
				- two
		- name: with_items
			debug:
				msg: "{{ item }}"
			with_items:
				- one
				- two
		- name: with_indexed_items
			debug:
				msg: "{{ item.0 }} - {{ item.1 }}"
			with_indexed_items:
				- one
				- two
```

### until

Repeats a task until a condition is met

- A keyword called ‘retries’ is available and it’s default value is 3
- A keyword called ‘delay’ is available and it’s default value is 5 (this implies seconds)

### Types of Loops

Standard Loops

- Iteration over simple lists

- Iteration over a list of hashes

  Hash items are dictionaries.

  Eg.

  `hash_item:`

  `name: ‘testuser1’, group: ‘wheels’`

  `name: ‘testuser2’, group: ‘root’`

  To call a particular value, you can use…

  [`item.name`](http://item.name) → calls ‘testuser1’

  [`item.group`](http://item.group) → calls ‘wheels’

- Iterating over a dictionary

  To iterate over a dictionary, it needs to be converted to a hash list first.

  Eg.

  `tag_data:`

  `Environment: dev`

  `Application: payment`

  Then do:

  `loop: “{{ tag_data | dict2items }}”`

  It essentially converts the dictionary to a hash list like this:

  loop:

  - `{ key: “Environment”, value: “dev” }`
  - `{ key: “Application”, value: “payment” }`

Complex Loops

Used to iterate over nested lists.

Inventory Loops

- `ansible_play_batch`

  Will use all the hosts that are there in the current batch of play execution

  Eg. `loop: “{{ ansible_play_batch }}`

- `groups[’all’]`

  Will use all the hosts that are there in the inventory

  Eg. `loop: “{{ groups[’all’] }}`

- `groups[’hostgroup’]`

  Will use all the hosts inside the hostgroup

  Eg. `loop: “{{ groups[’hostgroup’] }}`

  ![Screenshot 2023-01-07 at 16.48.52.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3f388637-918c-4fe6-bfac-f842c9672f4e/Screenshot_2023-01-07_at_16.48.52.png)

  ![Screenshot 2023-01-07 at 16.49.07.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/23f7c34c-15bd-40fb-924f-08a184274453/Screenshot_2023-01-07_at_16.49.07.png)

## Conditionals in Ansible

### When Statement

Eg.

```yaml
tasks:
  - name: shut down debian systems
    command: /sbin/shutdown -t now
    when: ansible_facts['os_family'] == "Debian"
```

### Loops & Conditionals

### Custom Facts

```yaml
---
- name: Perform version check
  hosts: localhost 
  gather_facts: no 
  vars: 
    my_version: 2.3.2

  tasks:
    - name: compare versions 1
      debug: 
        msg: "My version is higher than 2.0.0"
      when: my_version is version('2.0.0','>')
    - name: compare versions 2
      debug: 
        msg: "My version is higher than 3.0.0"
      when: my_version is version('3.0.0','>')
```

### Register Variables

```yaml
---
# Registering Variables

- hosts: localhost
  become: true
  tasks:
    - name: Get the hostname and register it to a variable
      command: hostname
      register: my_host
    - name: Print registered variable
      debug:
        msg: "The host name of the current target is {{ my_host.stdout }}"
```

### Register Variables with Loop

```yaml
---
# Registering variables with loop

- hosts: localhost
  become: true 
  tasks: 
    - name: Get the host directory contents
      command: ls /home
      register: my_home
    - name: print variable
      debug:
        msg: "my_home -> {{ my_home }}"
    - name: Print the home content list from registered variable
      debug:
        msg: "Home list -> {{ my_home.stdout_lines }}"
    - name: Print the home values in a loop
      debug:
        msg: "Content -> {{ item }}"
      loop: "{{ my_home.stdout_lines }}"
```

### Conditional Import

```yaml
---
- hosts: localhost
  become: true
  tasks:
    - name: Get OS Family
      debug:
      msg: "{{ ansible_facts['os_family'] }}"
    - name: Sample Message
      debug:
      when: ansible_facts['os_family'] == "Ubuntu"
---
- hosts: localhost
  become: true 
  tasks:
    - name: Get OS Distribution & Version
      debug:
        msg: "{{ ansible_facts['distribution'] }} - version {{ ansible_facts['distribution_major_version']}}"
    - name: Testing either CentOS 6 or Debian 7 or Ubuntu 20 systems
      command: echo "Either CentosOS 6 or Debian 7 or Ubuntu 20"
      when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7") or (ansible_facts['distribution'] == "Ubuntu" and ansible_facts['distribution_major_version'] == "20")
---
- hosts: localhost
  become: true 
  tasks:
    - name: Get OS Distribution & Version
      debug:
        msg: "{{ ansible_facts['distribution'] }} - version {{ ansible_facts['distribution_major_version']}}"
    - name: Testing Ubuntu 20 systems
      command: echo "Ubuntu 20 Systems"
      when: 
        - ansible_facts['distribution'] == "Ubuntu"
        - ansible_facts['distribution_major_version'] == "20"
```

## Ansible Handlers

- They are similar to tasks as in they are executed only when called/notified.
- If the notifying tasks remain unchanged, the handler will not be executed
- Uses the notify keyword
- Executed only once as a part of play execution & at the end of the play execution
- A single task can trigger multiple handlers
- A single handler can be notified by multiple tasks
- Each handler needs to have a globally unique name
- It is discouraged using variables/variable templates in handler names.

```yaml
# Example of Ansible Handlers
---
- hosts: localhost
  become: true 
  tasks:
    - name: task1
      command: hostname 
      notify:
        - my handler

  handlers:
    - name: my handler
      debug:
        msg: "executed my handler..."
# Example of a Playbook with multiple handlers
---
- hosts: localhost
  become: true
  tasks:
    - name: task1
      command: hostname
      notify:
        - my handler
        - my new handler
    - name: task2
      command: hostname
      notify:
        - my handler
    - name: task3
      command: hostname

  handlers:
    - name: my handler
      debug:
        msg: "executed my handler..."
    - name: my new handler
      debug:
        msg: "executed my NEW handler..."
```

### Controlling Handler Run

If you need to run handlers before end of play, you can use a task using the ‘meta’ module to flush handlers. `meta: flush_handlers`

This task will trigger any handlers that have been notified until this point in time of execution.

```yaml
---
- hosts: localhost
  become: true
  tasks:
    - name: task1
      command: hostname
      notify:
        - my handler3
    - name: task2
      command: hostname
      notify:
        - my handler2

    - name: Flush all handlers # All handlers notified by the tasks before this will be triggered
      meta: flush_handlers

    - name: task3
      command: hostname
      notify:
        - my handler1

  handlers:
    - name: my handler1
      debug:
        msg: "executed my handler11111..."
    - name: my handler2
      debug:
        msg: "executed my handler22222..."
    - name: my handler3
      debug:
        msg: "executed my handler33333..."
```

## Ansible Jinja2 Templating

- Template files are created separated and called into Ansible playbooks using the keyword ‘template’.
- Template files have the extension .j2
- Consist of 2 inputs - the template file and the data

### Tags in Jinja2

`{{ … }}` → variables for rendering process

`{% … %}` → to perform statement [executions.Eg](http://executions.Eg). for, if statements

Eg.

```python
my_list = ['one', 'two', 'three']

{% for item in my_list %}
	{{ item }}
{% endfor %}
```

`{# … #}` → used for commenting including multiline comments. You can also use a single `#` for line statements and `##` for inline comments.

Eg.

```python
my_list = ['one', 'two', 'three']

for item in my_list
	{{ item }} ## This is a comment and will be ignored.
endfor
```

### Jinja2 Filters

- Filters are used to modify variables or manipulate data
- Variables can be separated from filters by using the pipe symbol ‘|’
- Can be used to set default values (using the ‘default’ keyword)
- Different filters can be chained together

Eg. of using filters

```python
name = <b>john davis</b>

{{ name }} -> <b>john davis</b>
{{ name|striptags }} -> john davis # striptags is a filter
{{ name|striptags|title }} -> John Davis # This is how to chain filters 

# The 'default' filter - is used to prevent the program from returning an error if no variable has been declared.

My variable value is {{ myvar|default(0) }} # will return 0 if no variable has been defined.
My variable value is {{ myvar|default(" no name found ") # will return no name found instead of throwing an error
```

## Ansible Roles

- Use a predefined directory structure
- Are used to group vars, tasks, handlers, etc corresponding to an execution
- Make it easy to reuse and share configuration information
- Must be used within a playbook
- Are defined using YAML files with a predefined directory structure
- A role is a set of tasks to configure a host to serve a certain purpose
- Makes the reusability of codes easy for anyone for whom the role is suitable
- Are easily modifiable and therefore, reduce syntax errors

### Default Locations

```
/etc/ansible/roles
~/.ansible/roles
/usr/share/ansible/roles
```

### Ansible Role Structure

- Roles follow the same directory structure, meaning all roles follow this structure
- Name of the role is the same as the parent directory
- Roles directory should be at the same level as the playbook that calls the role
- Each directory will have a main.yaml/main.yml/main file.

### Standard Role Directories

| Name      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| defaults  | Default values for variables used in the role. Values have lowest priority. |
| vars      | Values for variables used in the role. Values have higher priority than defaults. |
| tasks     | Contains lists of tasks to be executed by the role.          |
| files     | Contains sources files that need to be copied to remote hosts. |
| templates | Contains the jinja2 template files used in the role.         |
| meta      | Contains the metadata of the role itself. Eg. author, supported platforms, name of the role, dependencies, etc. |
| handlers  | Stores the handlers which can be triggered by the notify directive from tasks. |
| library   | Modules which you want to use within the role.               |

### Using Roles

Roles can only be called inside a playbook

There are 3 ways to call Roles.

| Method        | Level    | Reuse   |
| ------------- | -------- | ------- |
| roles:        | Playbook | Static  |
| include_role: | Task     | Dynamic |
| import_role:  | Task     | Static  |

### Static vs Dynamic

Static

All import statements are pre-processed at the time when the playbook is parsed

Dynamic

All import statements are processed as they are encountered

### Dependency Implementation in Roles & Collections

- Using meta/requirements.yml file
- Using meta/main.yml and writing dependencies there

```python
---
roles:
 - name: <dependent_role_name>
	 version: <dependent_role_version>

collections
 - name: <dependent_collection_name>
   version: <dependent_collection_version>
   source: <https://galaxy.ansible.com/> or <your_own_source> # Eg. your Github repository
  
```

## Playbook Execution Order

1. `pre_tasks` → tasks that are executed before all the tasks, including roles.
2. `pre_task handlers` → handlers that are triggered by the pre-tasks
3. `roles` → listed in the roles section
4. `tasks` → tasks defined in the play in the order in which they are written
5. `handlers` → triggered by the roles and tasks
6. `post_tasks` → run after all other tasks including roles and handlers of those tasks and roles are executed
7. `post_task handlers` → handlers triggered by the post_tasks

### Multiple Executions of Roles

Roles are only executed once even if there are duplicates, except:

- The second instance of the role uses different parameters and is visible inside the playbook
- If the condition `allow_duplicates: true` is mentioned inside the meta section of the role

## Ansible Galaxy

- Create roles and collections
- Download roles created by community users

```python
ansible-galaxy init <role_name> --offline # Initialize a role offline

ansible-galaxy install <role_name> # Install a role from Ansible Galaxy website

ansible-galaxy list # List roles

ansible-galaxy remove <role_name> # Remove a role
```

### Galaxy Collections

Is a format for distribution and packaging of Ansible roles, playbooks, modules & plugins

- Metdata of galaxy collections are available in MANIFEST.json or galaxy.yml files

```python
ansible-galaxy collection install <collection_name>

ansible-galaxy collection remove <collection_name>
```

## Ansible Tags

- Execute or skip selected tasks
- Keyword `tags`
- Tags can be play level or tasks level. Play level tags are applied to all tasks.
- 2 Steps:
  - Add tags to your tasks
  - select or skip tasks based on tags when you run the playbook

```yaml
---
# Example playbook for working with tags

- name: working with tags
  hosts: localhost
  gather_facts: no
  tags:
    - playtag # this is a play level tag

  tasks:
    - name: Task A
      debug:
        msg: "Task A executed"
      tags: # these are tasks level tags
        - taska
        - groupdev

    - name: Task B
      debug:
        msg: "Task B executed"
      tags:
        - taskb
        - groupdev
        - customx

    - name: Task C
      debug:
        msg: "Task C executed"
      tags:
        - taskc
        - grouptest

    - name: Task D
      debug:
        msg: "Task D executed"
# Listing tags used in a playbook
ansible-playbook <playbook>.yml --list-tags

# Listing tasks used in a playbook
ansible-playbook <playbook>.yml --list-tasks

# Listing tasks based on tags in a playbook
ansible-playbook <playbook>.yml --list-tasks --tags <tagname>

# Listing tasks with skipped tags
ansible-playbook <playbook>.yml --list-tasks --skip-tags <tagname>

# Listing all tasks using multiple tags
ansible-playbook <playbook>.yml --list-tasks --tags "<tagname1>, <tagname2>" # shows all tasks with tags 'tagname1' or 'tagname2'

# Run tasks based on tags
ansible-playbook <playbook>.yml --tags <tagname> # Runs all tasks in the playbook with the tag 'tagname'

# Run all tasks with tags
ansible-playbook <playbook>.yml --tags tagged

# Run all tasks without tags
ansible-playbook <playbook>.yml --tags untagged
```

## Ansible Vault

- For encrypting files and variables
- All encrypted content will always include a tag `!vault`
- There can also be vault-id. It is a kind of a label. This is optional

### Operations of Ansible Vault

1. Encrypt an existing file or variable

   ```bash
   # Encrypting with Ansible Vault
   ansible-vault encrypt_string <password-source> '<string to encrypt>' --name '<variable name>'
   
   # Encrypting an existing file
   ansible-vault encrypt <filename> <filename> <filename>
   
   # Encrypting a file using a password file
   ansible-vault encrypt --vault-password-file <filename>
   
   # Encrypting a file using vault_id
   ansible-vault encrypt --vault-id <vault_id>@prompt <filename>
   
   # Encrypting a file using vault_id and password file
   ansible-vault encrypt --vault-id <vault_id_name>@<password-file-name> <filename>
   ```

2. Decrypt a file

   ```bash
   # Decrypt an existing file
   ansible-vault decrypt <filename>
   ```

3. View an encrypted file without breaking the encryption

   ```bash
   # Viewing an encrypted file
   ansible-vault view <filename>
   ```

4. Edit an encrypted file

   ```bash
   # Editing an encrypted file without decrypting it
   ansible-vault edit <filename>
   ```

5. Create an encrypted file

   ```bash
   # Creating an encrypted file
   ansible-vault create <filename>
   ```

6. Generate or reset encryption keys

   ```bash
   # Rekey an encrypted file
   ansible-vault rekey <filename>
   ```

### Encrypting with Ansible Vault

| !vault tag |      |
| ---------- | ---- |
| character  |      |
| —vault-id  |      |

### Use Encrypted Files for Ad-Hoc Commands

```python
ansible localhost -m <module> --ask-vault-pass
```

## Error Handling & Troubleshooting

### Rescue Block

- Rescue blocks define the tasks to run when an earlier task in a block fails.
- Ansible only runs rescue blocks when a task returns to a 'failed' state.
- Incorrect task definitions and unreachable hosts will not trigger the rescue block.

```yaml
---
- name: To check error handling with Ansible
  hosts: localhost 
  gather_facts: no 

  tasks:
    - name: Task to handle error
      block:
        - name: First Task
          debug:
            msg: 'First task executed successfully'

        - name: Failure tasks
          command: /bin/false

        - name: After Task
          debug:
            msg: 'Task after the failure task. Will this execute?'
			rescue:
        - name: Task printed when error
          debug:
            msg: 'Rescue task to the action now!!!'
```

### Always Block

```yaml
---
- name: To check error handling with Ansible
  hosts: localhost 
  gather_facts: no 

  tasks:
    - name: Task to handle error
      block:
        - name: First Task
          debug:
            msg: 'First task executed successfully'

        - name: Failure tasks
          command: /bin/false

        - name: After Task
          debug:
            msg: 'Task after the failure task. Will this execute?'
			rescue:
        - name: Task printed when error
          debug:
            msg: 'Rescue task to the action now!!!'
			
			always:
        - name: Task to execute always
          debug:
            msg: 'This task executes always'
---
- name: To check error handling with Ansible
  hosts: localhost 
  gather_facts: no 

  tasks:
    - name: TaskX
      block:
        - name: first task - TaskX
          debug:
            msg: 'First task executed successfully - TASKX'
            
    - name: TaskA
      block:
        - name: Failure task - TaskA
          command: /bin/false

			rescue:
				- name: This will rescue the entire playbook
					debug:
						msg: 'I rescued your shitty playbook'
            
    - name: TaskB
      block:
        - name: second task - TaskB
          debug:
            msg: 'Second task executed successfully - TASKB'

			rescue:
				- name: This is a redundant rescue task
					debug:
						msg: 'I will not be executed so dont waste time.'
```

### Troubleshooting

```yaml
---
# Sample YAML file to test troubleshooting in Ansible

- name: To perform some troubleshooting operation
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Task X
      debug:
        msg: 'Task X executed'

    - name: Task Y
      debug:
        msg: 'Task Y executed'
# Ways to Troubleshoot in Ansible

# Checking syntax before execution of playbook
ansible-playbook <playbook>.yml --syntax-check

# Checking a playbook with a dry run
ansible-playbook <playbook>.yml --check

# Additional information display in console
ansible-playbook <playbook>.yml --verbose

# Step-by-step execution of playbook
ansible-playbook <playbook>.yml --step
# > -N -> skip the task
# > -y -> execute the task and go to the next task
# > -c -> continue with remaining tasks without input (in short, cancelling the 'step' mode

# Start the execution of a playbook from a particular task
ansible-playbook <plabook>.yml --start-at-task="<taskname>"
```

### Debugger

Use the ‘debugger’ keyword to open the debugging window based on certain events.

```yaml
# Using the Debugger keyword to troubleshoot

---
# Sample YAML file to test troubleshooting in Ansible

- name: To perform some troubleshooting operation
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Task X
      debug:
        msg: 'Task X executed'

    - name: Task Y
      debug:
        msg: 'Task Y executed'
      debugger: always # debugger will always run after this task

# Other values for debugger
- debugger: never # debugger is never run
						on_reachable # if host is unreachable
						on_failed # if task fails
						on_skipped # if a task is skipped
```

## Dynamic Inventory

Inventory generated by other tools dynamically without specifying the hosts manually.

### Connectors

1. Inventory Plugins

   Recommended because they remain updated with the Ansible core and are customizable.

2. Inventory Scripts

### SSH Connection Variables

| ansible_host                 | ansible_ssh_host | give the name of the host for ansible to connect      |
| ---------------------------- | ---------------- | ----------------------------------------------------- |
| ansible_port                 | ansible_ssh_port | specify SSH port number (default is 22)               |
| ansible_user                 | ansible_ssh_user | specify SSH user                                      |
| ansible_password             | ansible_ssh_pass | SSH password to use                                   |
| ansible_ssh_private_key_file |                  | specify the path of the keyfile instead of a password |

### Privilege Escalation Variables

| ansible_become        | either ansible_sudo or ansible_su to force privilege escalation |
| --------------------- | ------------------------------------------------------------ |
| ansible_become_method | specify the method for privilege escalation                  |
| ansible_become_user   | either ansible_sudo_user or ansible_su _user (set the privilege escalation user) |
| ansible_become_pass   | either ansible_sudo_pass or ansible_su_pass (set the privilege escalation password) |

### Non-SSH Connectors

Using `ansible_connection:<connector>` to change the connection type.

- Local Connector → to deploy the playbook to the control server itself

- Docker Connector → to deploy the playbook directly to docker containers. Uses the local docker client.

  Variables:

  | ansible_host              | name of the docker connector to connect to                   |
  | ------------------------- | ------------------------------------------------------------ |
  | ansible_user              | user name to operate within the container (user must exist inside the container) |
  | ansible_become            | if ‘true’, can specify become_user to be used within the container |
  | ansible_docker_extra_args | pass additional arguments understood only by docker          |

## Ansible Tower

- Enterprise version of Ansible
- It has a hosted Ansible control server
- UI is available to monitor Ansible controls
- It was formerly named as AWX

### Features

- Ansible Tower Dashboard
- Logging of all automation activity
- Real-time job updates
- Multi-playbook workflows