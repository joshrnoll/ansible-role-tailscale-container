Role Name
=========

This role creates a docker container using the community.general.docker_container module along with a [Tailscale container](https://tailscale.com/blog/docker-tailscale-guide) to serve as it's networking namespace. Containers will be automatically added to your Tailnet using an Authkey or OAuth Client secret. 

Requirements
------------

You will need a [Tailscale Account](https://tailscale.com/) and a Tailscale [Auth key](https://tailscale.com/kb/1085/auth-keys) or [OAuth Client](https://tailscale.com/kb/1215/oauth-clients) secret for this role to run properly. It is recommended that you store your secret in Ansible vault, in a dictionary format like this:

```YAML
tailscale_containers_oauth_key:
  key: <random-secret-string-here>
```

You can reference this variable with:
```YAML
"{{ tailscale_containers_oauth_key[key] }}"
```

Ensure you use ```--ask-vault-pass``` or the -J flag when calling your playbook. 

The reason for the dictionary format is that I've found it plays nice with other Tailscale related roles that require this format. A simple key:value variable format should work for this role, but I recommend the dictionary format for consistency. 

Role Variables
--------------
```YAML
# Required variables with defaults
no_serve: false
https_container: false
public: false
serve_config: serve-config.json
extra_args: ""
userspace_networking: "true"

# Required variables without defaults
authkey: # Your Tailscale Auth key or Oath Client secret. It is recommended to use Ansible vault for this. 
service_name: # The container's hostname as it will be added to your tailnet. This will also be used to create a sub-directory on the host for bind mounts. This sub-directory will be in the {{ ansible_user }}'s home directory.
container_image: # Container's image (without tag appended)
container_tag: # Container's tag

# Optional variables 
container_user: # User to run the container as -- advanced use only (generally not used)
container_volumes: # Volumes in list format
env_vars: # Environment variables in dictionary format
container_labels: # Container labels in dictionary format
container_commands: # Commands to run on container startup
```

Dependencies
------------
**Docker** is required to be installed on the target node


Example Playbook
----------------

This example installs Uptime Kuma as a non-public (not exposed to the internet) container. It is not using userspace networking and has no environment variables provided
```
---
- name: Install Uptime Kuma
  hosts: uptime-kuma

  tasks:  
    - name: Install Uptime Kuma 
      include_role:
        name: ../../roles/ts-container
      vars:
        authkey: "{{ tailscale_containers_oauth_key['key'] }}"
        service_name: uptime-kuma
        container_image: louislam/uptime-kuma
        container_tag: latest
        serve_port: 3001
        userspace_networking: "false"
        public: false
        container_volumes:
          - /home/{{ ansible_user }}/{{ container_name }}/app:/app/data
```
This example installs plex as a public (exposed to the internet) container. It has several environment variables provided and is not using userspace networking. 
```
- name: Install plex
  hosts: plex

  tasks:
    - name: Include variables
      include_vars: 
        dir: ../../vars

    - name: Install plex
          include_role:
            name: ../../roles/ts-container
          vars:
            authkey: "{{ tailscale_containers_oauth_key['key'] }}"
            service_name: plex
            container_image: lscr.io/linuxserver/plex
            container_tag: arm64v8-latest
            https_container: true
            serve_port: 32400
            userspace_networking: "false"
            public: true
            container_volumes:
              - /home/{{ ansible_user }}/plex/config:/config
              - /home/{{ ansible_user }}/media/media:/media
            env_vars:
              PUID: "1000"
              PGID: "999"
              TZ: "America/New_York"
              Version: "docker"
```

License
-------

BSD

Author Information
------------------

Josh Noll
https://www.joshrnoll.com
