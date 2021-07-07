# Sử dụng Ansible để tạo nhiều Template trên AWX

Vì việc tạo các tác vụ phục vụ cho quá trình vận hành có thể lên đến hàng trăm template nên ta không thể tạo bằng tay trên giao diện awx 

Để sử dụng Ansible tương tác với AWX ta sẽ sử dụng module [tower_job_template](https://docs.ansible.com/ansible/2.9/modules/tower_job_template_module.html)

***!Lưu ý:*** Việc sử dụng module này phụ thuộc vào phiên bản Ansible bạn sử dụng vì sẽ có sự khác nhau về tên của paramenters giữa các phiên bản.

Để có thể sử dụng được module `tower_job_template` ta sẽ phải cài `ansible-tower-cli`

Đây là một phần mềm cung cấp bộ công cụ lệnh cho Ansible-Tower. Nó cho phép sử dụng Ansible-Tower thông qua lệnh UNIX hoặc cũng có thể sử dụng làm thư viện client cho các ứng dụng python và là sự liên kết phát triển tương tác API thông qua REST-API của Ansible-Tower.

## Hướng dẫn sử dụng Tower-cli

- Cài đặt bằng cách:

```sh
pip3 install ansible-tower-cli
```
- Để thiết lập môi trường awx đã có ta cấu hình file `~/.tower_cli.cfg` như sau:
```ini
[general]
username = admin
host = https://somethingcool/
verify_ssl = false
password = myansibleawx
certificate = /etc/ssl/certs/awx.pem
```
- Dùng lệnh để hiển thị các thông tin trên awx:
```sh
[root@deployment ansible-openstack]# tower-cli  user list
== ======== ============== ========== ========= ============ =================
id username     email      first_name last_name is_superuser is_system_auditor
== ======== ============== ========== ========= ============ =================
 2 admin    root@localhost                              true             false
== ======== ============== ========== ========= ============ =================
```

## Sử dụng Ansible tạo templates trên AWX

- Tạo file inventory với nội dung là thông tin node cài đặt awx:
```ini
[awx]
deployment ansible_host=somethingcool ansible_port=22
```

- Tạo file playbook:
```yaml
---
- hosts: all
  vars:
     - list_host:
             - controller01
             - controller02
             - controller03
     - list_service:
             - openstack-nova-api
             - openstack-nova-scheduler
             - openstack-nova-conductor
             - openstack-nova-novncproxy
             - openstack-glance-api
  tasks:
   - name: Test module awx
     tower_job_template:
        name: "OPS_HCM_{{ item[0] }}_restart_{{ item[1] }}"
        validate_certs: yes
        job_type: "run"
        inventory: "OPS_Inventory"
        project: "Openstack_Ansible"
        playbook: "operate-playbooks/restart_service.yml"
        credential: "Ceph1_key"
        state: "present"
        limit: "{{ item[0] }}"
        extra_vars_path: "/var/lib/awx/projects/ansible-openstack/vars/custom_{{ item[1] }}_extra_vars_customise_other_site.yml"
     with_nested:
              - "{{ list_host }}"
              - "{{ list_service }}"
```
Playbook này sẽ tạo ra 15 templates: Restart một service trong `list_service` trên từng node trong `list_host`


