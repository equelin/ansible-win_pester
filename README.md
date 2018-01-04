# ansible-win_pester

> This module is under developpement. Tested with ANsible 2.4 and Windows Server 2016. Please open issues or submit PR.

Run [Pester](https://github.com/pester/Pester) tests with Ansible on Windows hosts.

You may integrate this module in your Ansible workflow to check if the configuration of your Windows host is correct after running your playbook. 

## Requirements

- The Pester module has to be available on the remote host.
- The test files have to be availaible on the remote host. You can use the `win_copy` module to copy the test files.

## Instructions

Copy the files `win_pester.ps1` and `win_pester.py` into a folder available in your Ansible module path.

Alternatively you can store the files in a folder named `library` alongside your playbook.

## Options

| parameter | required | default | choices | comments |
| --- | --- | --- | --- | --- |
| src | true | | | (string) Specifies the source path of the test files on the remote host. Could be files or folder. If a folder, the module will try to run Pester tests with all ps1 files available in it.|
| version | false | | | (string) Specifies the minimum version of the Pester module that has to be available on the remote host. |

## Examples

```yaml
- name: Run Pester test(s) in the file C:\Pester\test01.test.ps1
  win_pester:
    src: C:\Pester\test01.test.ps1

- name: Run Pester test(s) located in the 'C:\Pester' folder and defining the minimum version of the pester module
  win_pester:
    src: C:\Pester
    version: '3.4.0'
```

## Return Values

| name | description | returned | type | sample |
| --- | --- | --- | --- | --- |
| pester_result | Data returned by the Pester module | success |  |   |
| pester_version | Version of the Pester module find on the remote host | | string | '4.1.0' |

## Playbook

This playbook will copy the Pester tests files on the remote hosts, run the tests and verified if a test failed. The tests files are deleted at the end.

```yaml
- name: Test out win_pester module
  hosts: windows
  vars:
    local_test_files:
      - "files/test01.test.ps1"
      - "files/test02.test.ps1"
    remote_test_folder: C:\Pester\

  tasks:
    - name : Copy test file(s)
      win_copy:
        src: "{{ item }}"
        dest: "{{ remote_test_folder }}"
      with_items: "{{local_test_files}}"

    - name: Run Pester test(s) located in a folder and defining the minimum version of the pester module
      win_pester:
        src: "{{ remote_test_folder }}"
        version: '3.4.0'
      register: result

    - name: Check if one of the Pester tests failed
      assert:
        that: result.pester_result.FailedCount == 0

    - name: Delete test folder
      win_file:
        path: "{{ remote_test_folder }}"
        state: absent
```

## Developpement

Feel free to open issues and submit PR.

If you want to run integration tests, you can look at the Ansible role in `./test/integration/targets/win_pester`

## Author

**Erwan Quélin**
- <https://github.com/equelin>
- <https://twitter.com/erwanquelin>

## License

GNU General Public License v3.0

See [LICENSE](LICENSE) to see the full text.