# Playbook STIG RHEL 6.5

Install the role from Ansible Galaxy by using the galaxy.yml file in this repo.

To install this role execute the following from the commandline.

```
ansible-galaxy install -r galaxy.yml
```

Inside the file it pulls the role `nousdefions.STIG-RHEL6` from Galaxy and renames the file `STIG-RHEL6` in the `roles` folder. 

```
# from galaxy
- src: nousdefions.STIG-RHEL6
  name: STIG-RHEL6
```

# Ansible Tower Offline Usage

Install the role to 

```
/var/lib/awx/projects/LOCAL-RHEL6.5/roles/STIG-RHEL6/
```

### By default this playbook remediates `CAT 1` and skips `CAT 2 + 3`. To enable `CAT 2` add the following to `Extra Vars`. 

```
rhel6stig_cat2: true
```

In Tower when you launch the Job Template add the following skip tags to disable any tasks that make out bound calls. 

### Skip Tags:

```
V-38476,V-38481,V-38496,V-38483,V-38489,V-38519,V-51363
```




Here is more detail on the exact tasks you are skipping.

# CAT 1 Skip Tags
V-38476

# CAT 2 Skip Tags
V-38481
V-38496
V-38483
V-38489
V-38519
V-51363


---------------

## CAT 1

```
- name: V-38476 High  Vendor-provided cryptographic certificates must be installed to verify the integrity of system software.
  rpm_key: state=present key={{ gpg_key_url }}
  tags: [ 'cat1' , 'V-38476' , 'rpm' ]
```

## CAT 2

```
- name: Checking for updates
  command: yum check-update
  changed_when: result.rc == 100
  failed_when: result.rc == 1
  register: result
  tags: [ 'cat2' , 'V-38481' , 'yum' , 'updates' ]


- name: V-38481 Medium  System security patches and updates must be installed and up-to-date
  yum: name=* state=latest
  when: rhel6stig_update_packages
  tags: [ 'cat2' , 'V-38481' , 'yum' , 'updates' ]

- name: "V-38496 Medium  Default system accounts, other than root, must be locked"
  command: passwd -l {{ item }}
  with_items: system_users.stdout_lines
  tags: [ 'cat2' , 'V-38496' , 'accounts' ]

# Get a list of all repo files on remote system
# Ideally, this would be done using with_fileglobs instead of two separate actions,
# but with_fileglobs only operates on the host filesystem, not the remote.
- name: List yum repository files
  shell: ls /etc/yum.repos.d/*.repo
  changed_when: false
  register: yum_repos
  failed_when: "'FAIL' in yum_repos.stdout"
  tags: [ 'cat2' , 'V-38483' , 'V-38487' , 'rpm' , 'gpgcheck' ]

- name: "V-38483 Medium  The system package management tool must cryptographically verify the authenticity of system software packages during installation\n     V-38487  Low     The system package management tool must cryptographically verify the authenticity of all software packages during installation"
  replace: backup=yes dest={{ item }} regexp='^gpgcheck=0' replace='gpgcheck=1'
  tags: [ 'cat2' , 'V-38483' , 'V-38487' , 'rpm' , 'gpgcheck' ]
  with_flattened:
    - /etc/yum.conf
    - yum_repos.stdout_lines

    minute={{ rhel6stig_aide_cron["aide_minute"] | default('05') }}
    hour={{ rhel6stig_aide_cron["aide_hour"] | default('04') }}
    day={{ rhel6stig_aide_cron["aide_day"] | default('*') }}
    month={{ rhel6stig_aide_cron["aide_month"] | default('*') }}
    weekday={{ rhel6stig_aide_cron["aide_weekday"] | default('*') }}
    job="{{ rhel6stig_aide_cron["aide_job"] }}"
  tags: [ 'cat2' , 'V-38670' , 'V-38673' , 'V-38695' , 'V-38696' , 'V-38698' , 'V-38700' , 'file_integrity' , 'aide' ]

- name: V-38489 Medium  A file integrity tool must be installed
  yum: name=aide state=present
  tags: [ 'cat2' , 'V-38489' , 'file_integrity' , 'aide' ]
  notify: init aide

- name: V-51363 Medium  The system must use a Linux Security Module configured to enforce limits on system services.
  selinux: conf=/etc/selinux/config policy={{ rhel6stig_selinux_pol }} state=enforcing
  tags: [ 'cat2' , 'V-51363' , 'selinux' ]

# Added to correct file
- name: V-38615 Medium  The SSH daemon must be configured with the Department of Defense (DoD) login banner.
  lineinfile:
    path: /etc/ssh/sshd_config
    line: "        Banner /etc/issue"
```
