# Домашнее задание к занятию "08.04 Создание собственных modules"

## Подготовка к выполнению
1. Создайте пустой публичных репозиторий в любом своём проекте: `my_own_collection`
2. Скачайте репозиторий ansible: `git clone https://github.com/ansible/ansible.git` по любому удобному вам пути
3. Зайдите в директорию ansible: `cd ansible`
4. Создайте виртуальное окружение: `python3 -m venv venv`
5. Активируйте виртуальное окружение: `. venv/bin/activate`. Дальнейшие действия производятся только в виртуальном окружении
6. Установите зависимости `pip install -r requirements.txt`
7. Запустить настройку окружения `. hacking/env-setup`
8. Если все шаги прошли успешно - выйти из виртуального окружения `deactivate`
9. Ваше окружение настроено, для того чтобы запустить его, нужно находиться в директории `ansible` и выполнить конструкцию `. venv/bin/activate && . hacking/env-setup`

## Основная часть

Наша цель - написать собственный module, который мы можем использовать в своей role, через playbook. Всё это должно быть собрано в виде collection и отправлено в наш репозиторий.

1. В виртуальном окружении создать новый `my_own_module.py` файл
2. Наполнить его содержимым:
```python
#!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type
import os.path
DOCUMENTATION = r'''
---
module: my_test

short_description: This is my test module

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: Test module for writing file with content to path.

options:
    path:
        description: Path to new file with name.
        required: true
        type: str
    content:
        description: contetnt wich would be written to file on PATH
        required: true
        type: str
# Specify this value according to your collection
# in format of namespace.collection.doc_fragment_name
extends_documentation_fragment:
    - my_netology.my_collection.my_own_module

author:
    - Bazhin D.S. (https://github.com/homopoluza)
'''

EXAMPLES = r'''
# Pass in a message
- name: Write a file
  my_namespace.my_collection.my_test:
    path: '/tmp/file.txt'
    content: 'some content'

'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
original_message:
    description: The original content that was passed in.
    type: str
    returned: always
    sample: 'some content'
message:
    description: The output message that the test module generates.
    type: str
    returned: always
    sample: 'file is created'
'''

from ansible.module_utils.basic import AnsibleModule


def run_module():
    # define available arguments/parameters a user can pass to the module
    module_args = dict(
        name=dict(type='str', required=True),
        new=dict(type='bool', required=False, default=False)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # changed is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False,
        original_message='',
        message=''
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        module.exit_json(**result)

    if not os.path.exists(module.params['path']):
        file1 = open(module.params['path'],"w")
        file1.write(module.params['content'])
        file1.close()
        result['changed'] = True
        result['message'] = 'file is created'
    else:
        result['changed'] = False    
        result['message'] = 'file exists'
    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    result['original_message'] = module.params['content']

    # in the event of a successful module execution, you will want to
    # simple AnsibleModule.exit_json(), passing the key/value results
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```
Или возьмите данное наполнение из [статьи](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module).

3. Заполните файл в соответствии с требованиями ansible так, чтобы он выполнял основную задачу: module должен создавать текстовый файл на удалённом хосте по пути, определённом в параметре `path`, с содержимым, определённым в параметре `content`.
4. Проверьте module на исполняемость локально.

```
(venv) vagrant@vagrant:~/ansible$ python3 -m ansible.modules.my_own_module input.json

{"invocation": {"module_args": {"content": "some content", "path": "/tmp/file.txt"}}, "message": "file is created", "changed": true, "original_message": "some content"}
```
5. Напишите single task playbook и используйте module в нём.

```
(venv) vagrant@vagrant:~/ansible$ ansible-playbook test.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the
implicit localhost does not match 'all'
[WARNING]: ansible.utils.display.initialize_locale has not been called, this may result
in incorrectly calculated text widths that can cause Display to print incorrect line
lengths

PLAY [test my module] *******************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [localhost]

TASK [run my module] ********************************************************************
changed: [localhost]

TASK [dump test_out] ********************************************************************
ok: [localhost] => {
    "msg": {
        "changed": true,
        "failed": false,
        "message": "file is created",
        "original_message": "some content"
    }
}

PLAY RECAP ******************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

6. Проверьте через playbook на идемпотентность.

```
(venv) vagrant@vagrant:~/ansible$ ansible-playbook test.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'
[WARNING]: ansible.utils.display.initialize_locale has not been called, this may result in incorrectly
calculated text widths that can cause Display to print incorrect line lengths

PLAY [test my module] **********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [localhost]

TASK [run my module] ***********************************************************************************
ok: [localhost]

TASK [dump test_out] ***********************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "message": "file is created",
        "original_message": "some content"
    }
}

PLAY RECAP *********************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
7. Выйдите из виртуального окружения.
8. Инициализируйте новую collection: `ansible-galaxy collection init my_own_namespace.my_own_collection`

```
vagrant@vagrant:~/ansible/col$ ansible-galaxy collection init my_netology.my_collection
- Collection my_netology.my_collection was created successfully
```
9. В данную collection перенесите свой module в соответствующую директорию.
10. Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть default всех параметров module
11. Создайте playbook для использования этой role.
12. Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег `1.0.0` на этот коммит.
13. Создайте .tar.gz этой collection: `ansible-galaxy collection build` в корневой директории collection.
14. Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.
15. Установите collection из локального архива: `ansible-galaxy collection install <archivename>.tar.gz`
16. Запустите playbook, убедитесь, что он работает.

```
vagrant@vagrant:~/ansible/col$ ansible-playbook site.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'

PLAY [test my module] **********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [localhost]

TASK [run my module] ***********************************************************************************
ok: [localhost]

TASK [dump test_out] ***********************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "message": "file exists",
        "original_message": "some content"
    }
}

PLAY RECAP *********************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

vagrant@vagrant:~/ansible/col$ cat site.yml 
---
  - name: test my module
    hosts: localhost
    tasks:
    - name  : run my module
      my_netology.my_collection.my_own_module:
        path: "/tmp/file.txt"
        content: "some content"
      register: test_out
    - name: dump test_out
      debug:
        msg: "{{ test_out }}"  
```
17. В ответ необходимо прислать ссылку на репозиторий с collection

```
https://github.com/homopoluza/my_own_collection
```