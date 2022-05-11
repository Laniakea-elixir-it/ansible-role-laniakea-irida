laniakea.irida
=========

This role installs and configure Galaxy and the IRIDA web interface using as reference the [IRIDA documentation](https://phac-nml.github.io/irida-documentation/).
Galaxy is installed using the [phac-nml docker image](https://hub.docker.com/r/phacnml/galaxy-irida-20.09).
Two configurations are possible:
* Galaxy and IRIDA on the same VM: this is obtained by setting `type_of_node: all` and both `irida_server_ip` and `galaxy_wn_ip` to the same IP. This configuration has been tested on Ubuntu 20.04
* Galaxy and IRIDA on two VMs: this is obtained by running the role on each VM, setting `type_of_node` to `galaxy` and `irida` respectively. When this configuration is set, NFS is used to share data between Galaxy and IRIDA, stored in the export directory. This configuration has been tested on Ubuntu 20.04 for IRIDA and CentOS7 for Galaxy.


Role Variables
--------------

## VM configuration variables
| Variable           | Description                              | Default |
| ------------------ | ---------------------------------------- | ------- |
| type_of_node | Possible values are 'irida', 'galaxy' or 'all', see above for explanation | all |
| irida_server_ip | IP of the VM hosting IRIDA | '' |
| galaxy_wn_ip | IP of the VM hosting Galaxy | '' |
| export_dir | Export directory where data and docker images are stored | /export |
| irida_dir | Directory where IRIDA data is stored. Also the NFS mountpoint when using two VMs | '{{ export_dir }}/irida' |
| nfs_root | NFS root | "/srv/nfs" |
| irida_nfs_dir | Shared NFS directory | "{{ nfs_root }}/irida" |
| galaxy_storage_dir | Directory where the Galaxy docker volume is mounted | '{{ export_dir }}/galaxy' |
| docker_dir | Directory where docker images are stored | '{{ galaxy_storage_dir }}/docker-images' |

## Galaxy docker variables
| Variable           | Description                              | Default |
| ------------------ | ---------------------------------------- | ------- |
| galaxy_url | Galaxy URL | "{{ 'localhost' if irida_server_ip==galaxy_wn_ip else galaxy_wn_ip }}" |
| galaxy_port | Port where Galaxy is served | "48888" |
| galaxy_docker_name | Name of the Galaxy docker | galaxy-irida-20.09 |
| galaxy_docker_image | Full name of the Galaxy docker image | phacnml/galaxy-irida-20.09 |
| docker_ports_mapping | Docker ports mapping | "{{ galaxy_port }}:80" |
| galaxy_central_dir | Docker galaxy-central directory | /export/galaxy-central |
| host_galaxy_central_dir | Host galaxy-central directory | "{{ galaxy_storage_dir}}/galaxy-central" |
| docker_volume_mapping | Docker volumes mapping | ["{{ irida_dir }}:{{ irida_dir }}", "{{ host_galaxy_central_dir }}:{{ galaxy_central_dir }}"] |
| galaxy_venv | Path to docker Galaxy venv | /galaxy_venv |

## IRIDA variables
| Variable           | Description                              | Default |
| ------------------ | ---------------------------------------- | ------- |
| irida_url | IRIDA URL | "{{ irida_server_ip }}" |
| irida_port | Port where IRIDA is served | "8080" |
| irida_data | IRIDA data directory | "{{ irida_dir }}/data" |
| irida_sequencing | IRIDA sequencing directory | "{{ irida_data }}/sequencing" |
| irida_reference | IRIDA reference directory | "{{ irida_data }}/reference" |
| irida_analysis | IRIDA analysis directory | "{{ irida_data }}/analysis" |
| irida_assembly | IRIDA assembly directory | "{{ irida_data }}/assembly" |
| irida_snapshot | IRIDA snapshot directory | "{{ irida_data }}/snapshot" |
| irida_conf_dir | IRIDA config directory | "/etc/irida" |
| irida_analytics | IRIDA analytics directory | "{{ irida_conf_dir }}/analytics" |
| irida_plugins | IRIDA plugins directory | "{{ irida_conf_dir }}/plugins" |
| irida_conf_url | URL to the sample irida.conf file | "https://phac-nml.github.io/irida-documentation/administrator/web/config/irida.conf" |
| irida_web_conf_url | URL to the sample web.conf file | https://phac-nml.github.io/irida-documentation/administrator/web/config/web.conf |
| irida_war_url | URL to the IRIDA .war file | https://github.com/phac-nml/irida/releases/download/22.01.1/irida-22.01.1.war |
| irida_user | IRIDA user in Galaxy | irida |
| irida_email | IRIDA mail in Galaxy | irida@local.localhost |
| irida_password | IRIDA password in Galaxy | irida01 |

## irida_conf_params
Variables stored in the `irida_conf_params` variable, used to template the irida.conf file. Refer to the [IRIDA documentation](https://phac-nml.github.io/irida-documentation/administrator/web/) for the meaning of each variable.
| Variable           | Default |
| ------------------ | ------- |
| sequence_file_base_directory| "{{ irida_sequencing }}" |
| reference_file_base_directory "{{ irida_reference }}" |
| output_file_base_directory | "{{ irida_analysis }}" |
| assembly_file_base_directory | "{{ irida_assembly }}" |
| snapshot_file_base_directory | "{{ irida_snapshot }}" |
| pipeline_plugin_path | "{{ irida_plugins }}" |
| file_processing_core_size | 4 |
| file_processing_max_size | 4 |
| file_processing_queue_capacity | 512 |
| file_processing_process | "true" |
| spring_datasource_url | jdbc:mysql://127.0.0.1:3306/irida |
| spring_datasource_username | irida |
| spring_datasource_password | irida |
| galaxy_execution_apiKey | xxxx |
| galaxy_execution_email | "{{ irida_email }}" |
| galaxy_library_upload_timeout | 3600 |
| irida_workflow_max_running | 4 |
| irida_workflow_analysis_threads | 4 |

## irida_web_conf_params
Variables stored in the irida_web_conf_params variable. Also in this case, refer to the IRIDA documentation for the meaning of each variable.
| Variable           | Default |
| ------------------ | ------- |
| server_base_url | "http://127.0.0.1:{{ irida_port }}" |
| mail_server_host | your-mail-server.local |
| mail_server_protocol | smtp |
| mail_server_email | irida@your-mail-server.local |
| mail_server_username | IRIDA Platform |
| mail_server_password | "" |
| mail_server_port | 25 |

## Tomcat8 variables
| Variable           | Description                              | Default |
| ------------------ | ---------------------------------------- | ------- |
| tomcat_conf_file | Tomcat8 configuration file path | "/etc/default/tomcat8" |
| tomcat_conf_params.JAVA_HOME | JAVA_HOME variable written in the tomcat conf file | "/usr/lib/jvm/java-11-openjdk-amd64/" |
| tomcat_conf_params.JAVA_OPTS | JAVA_OPTS variable written in the tomcat conf file | "-Dspring.profiles.active=prod -Dirida.db.profile=prod -Djava.awt.headless=true -XX:+UseConcMarkSweepGC -Xms1024m -Xmx1024m -XX:+UseConcMarkSweepGC" |
| tomcat_webapps | Tomcat8 webapps file path | "/var/lib/tomcat8/webapps" |

## NGINX variables
| Variable           | Description                              | Default |
| ------------------ | ---------------------------------------- | ------- |
| client_max_body_size | NGINX variable, set to 0 in order to avoid upload limits | 0 |


Dependencies
------------

This role uses the [indigo-dc.docker](https://galaxy.ansible.com/indigo-dc/docker).

Example Playbook
----------------

To configure IRIDA and Galaxy in the same VM:
```yml
---
- name: IRIDA + Galaxy on the same VM
  hosts: all
  roles:
    - role: laniakea.irida
      vars:
        type_of_node: 'all'
        irida_server_ip: 'IP'
        galaxy_wn_ip: 'same_IP'
      become: true
```

To configure IRIDA:
```yml
---
- name: Configure IRIDA
  hosts: all
  roles:
  - role: laniakea.irida
    vars:
      type_of_node: 'irida'
      irida_server_ip: 'IRIDA_IP'
      galaxy_wn_ip: 'Galaxy_IP'
```

To configure Galaxy:
```yml
---
- name: Configure Galaxy
  hosts: all
  roles:
  - role: laniakea.irida
    vars:
      type_of_noe: 'galaxy'
      irida_server_ip: 'IRIDA_IP'
      galaxy_wn_ip: 'Galaxy_IP'
```

License
-------


Author Information
------------------

[Daniele Colombo](https://github.com/dacolombo), [Marco Antonio Tangaro](https://github.com/mtangaro)