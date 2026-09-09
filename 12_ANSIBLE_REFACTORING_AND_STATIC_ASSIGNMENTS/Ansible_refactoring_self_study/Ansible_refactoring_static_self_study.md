# Ansible Artifact Re-Use

## Introduction

Ansible is an open-source automation tool that simplifies various IT processes such as configuration management, application deployment, and task automation. One of the key features of Ansible is the ability to reuse artifacts, which enhances efficiency, reduces redundancy, and ensures consistency across different playbooks and roles. 

This documentation provides a detailed guide on how to effectively reuse artifacts in Ansible, based on the principles and practices outlined in the [official Ansible documentation](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse.html). 

## Concepts

### 1. Roles

`Roles` are a collection of tasks, variables, files, templates, and modules that can be reused across different playbooks. They help organize playbooks by encapsulating the necessary components for a particular functionality. 

roles/  
└── example_role/  
&emsp;&emsp;&emsp;├── tasks/    
    &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── main.yml  
    &emsp;&emsp;&emsp;├── handlers/  
       &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── main.yml  
    &emsp;&emsp;&emsp;├── files/  
    &emsp;&emsp;&emsp;├── templates/  
    &emsp;&emsp;&emsp;├── vars/  
   &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── main.yml  
    &emsp;&emsp;&emsp;├── defaults/  
       &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── main.yml  
    &emsp;&emsp;&emsp;├── meta/  
       &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── main.yml  
    &emsp;&emsp;&emsp;└── tests/  
        &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;├── inventory  
        &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;└── test.yml



### 2. Includes and Imports

`Includes` and `imports` are mechanisms to incorporate external content into a playbook. They help split complex playbooks into manageable pieces, enhancing readability and maintainability. Refactoring between these two mechanisms impacts how tasks execute, how loops behave, and how Ansible handles tags and properties. 

* **Include (Dynamic):** Dynamically includes `tasks`, `roles`, or other `playbooks` during the execution of a playbook (at runtime). Ansible processes these `tasks` only when it encounters them during the play.
* **Import (Static):** Statically inserts `tasks`, `roles`, or other `playbooks` at the time the playbook is parsed (before execution begins). Pre-parsing means the imported `tasks` become an integral part of the main playbook structure.

### 3. Variables

`Variables` in Ansible allow for dynamic content and flexible playbooks. They can be defined in various places such as inventory files, playbooks, roles, or external variable files. Variable precedence determines which value wins when the same variable is defined in multiple places. 

### 4. Templates

`Templates` in Ansible use the `Jinja2` templating engine to generate files dynamically based on variables and other data. Templates can be reused across different `roles` and `playbooks` to enforce configuration standards across varying environments. 

### 5. Dynamic Includes

`Dynamic includes` enable the inclusion of content based on specific conditions evaluated during runtime. This allows for highly flexible and conditional playbook execution paths based on real-time system facts. 

## Implementing Reuse in Ansible

### Using Roles

* **Creating a Role:** To create a role skeleton structure, use the ansible-galaxy command: 

```bash

    ansible-galaxy init example_role
```  

* **Using a Role Statically in a Playbook:** 

```yaml

- hosts: webservers
  roles:
    - example_role
```


### Using Includes and Imports

* **Including a Task File (Dynamic):** Evaluated at runtime. Ideal when task file paths depend on variables discovered during execution. 

```yaml

- include_tasks: tasks/example_tasks.yml
```

* **Importing a Task File (Static):** Evaluated at parse time. Mandatory if you need to view or target specific inner tasks with command-line tags before the play runs. 

```yaml

- import_tasks: tasks/example_tasks.yml
```

* **Including a Role Dynamically:** Allows roles to be executed conditionally or inside a loop. 

```yaml

- include_role:
    name: example_role
```


### Using Variables

* **Defining Variables in a Playbook:** 

```yaml

- hosts: webservers
  vars:
    http_port: 80
```

* **Using External Variable Files:** 

```yaml

- hosts: webservers
  vars_files:
    - vars/external_vars.yml
```

### Using Templates

* **Creating a Template:** Create a Jinja2 template file, for example `templates/example_template.j2`: 

```nginx

server {
    listen {{ http_port }};
    server_name {{ server_name }};
}
```

* **Using a Template in a Playbook:** 

```yaml

- name: Apply web server configuration
  template:
    src: templates/example_template.j2
    dest: /etc/nginx/sites-available/example
```


### Using Dynamic Includes

* **Conditionally Including a Task:** 

```yaml

- include_tasks: tasks/example_tasks.yml
  when: ansible_facts['os_family'] == "Debian"
```


## Best Practices

* **Modularize Playbooks:** Break down complex playbooks into smaller, reusable components using roles, includes, and imports to minimize blast radiuses and duplicate code.
* **Understand the Trade-offs of Static vs. Dynamic Refactoring:** 

  * **Loops:** You *cannot* use a loop or with_items on a static import_tasks statement. If you need to loop over a block of tasks, refactor it to use include_tasks.
  * **Tags & Inheritances:** Applying a tag to an import_tasks statement applies that tag to all child tasks inside the file. Applying a tag to include_tasks only applies it to the inclusion task itself, not the tasks within.
  * **Handlers:** If a task inside an imported file acts as a handler, it can be notified. Handlers inside dynamic includes cannot be easily targeted by name from outside the include block.
* **Use Variables Efficiently:** Define default fallback values in defaults/main.yml within roles, and override them at appropriate higher-precedence levels (like group_vars or extra vars) to keep configurations clean.
* **Template Standardization:** Create standard, abstract templates that rely on strict variable schemas so they can safely span multiple environments (Dev, Staging, Prod).
* **Document Reusable Components:** Ensure that all reusable components feature a clear README.md containing expected variables, dependencies, and execution examples to facilitate seamless collaboration.

## Conclusion

Reusing artifacts in Ansible is a powerful way to enhance automation efficiency and maintainability. By leveraging `roles`, `includes`, `imports`, `variables`, `templates`, and `dynamic includes`, you can create modular, scalable, and reusable playbooks. 

Following the best practices outlined in this guide—especially when deciding between static parsing and dynamic execution—will help you get the most out of Ansible's capabilities for artifact reuse.