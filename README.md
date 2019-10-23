# Ansible recipes and gotchas

## Process will be terminated with RECEIVED SIGNAL 1: SIGHUP after playbook finishes
While creating a role for Hadoop deployment I discovered that the NameNode process always shut down after the playbook finished with the message `ERROR org.apache.hadoop.hdfs.server.namenode.NameNode: RECEIVED SIGNAL 1: SIGHUP`. The reason for this is that the NameNode process registered a UNIX signal handler for HUP (HangUp) which is send to the process as soon as the ssh connection create by ansible is closed. To avoid this signal to be sent `nohup` can be used in front of any command. In the case of the Hadoop NameNode I've ended up using the following command to start it: `nohup hdfs --daemon start namenode`

## Creating a numbered, comma seperated string from a list of hosts

```
# This will create the string "nn1,nn2,nn3,..." from a list of nodes

- name: generate namenode list 1/3
  set_fact: namenodes="nn1"

- name: generate namenode list 2/3
  set_fact: namenodes="{{namenodes + ',nn' + (idx+2)|string }}"    
  loop: "{{ groups['hadoop_namenodes'][1:] }}"
  loop_control:
    index_var: idx
```

## Advanced usage of the xml module

Use the short snippet below to create a XML structure with multiple equal named properties like this:

```
<configuration>
  <property>
    <name>ke1y</name>
    <value>value1</value>
  </property>
  <property>
    <name>key2</name>
    <value>value2</value>
  </property>
</configuration>
```

```
- name: check if element exists
  xml:
    path: "{{path}}"
    xpath: /configuration/property[name="{{name}}"]
    count: yes
  register: hits

- name: create <property> if not exists
  xml:
    path: "{{path}}"
    xpath: /configuration
    add_children:
      - property: null
    state: present
  when: hits.count == 0

- name: create property '{{name}}' in '{{path}}' if not exists
  xml:
    path: "{{path}}"
    xpath: /configuration/property
    add_children:
      - name: "{{name}}"
    state: present
  when: hits.count == 0

- name: set property '{{name}}' to '{{value}}' in {{path}}
  xml:
    path: "{{path}}"
    xpath: /configuration/property[name="{{name}}"]/value
    value: "{{value}}"
    state: present
```
