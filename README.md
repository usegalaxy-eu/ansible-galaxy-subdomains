# ansible-galaxy-subdomains

Sets up the subdomain static content and themes.  
(not fully serious) [Presentation on the topic](https://docs.google.com/presentation/d/1N1u1b0AqJ-ZVz8qKO5YxkwUSjSR5HwnuSxTsj6wAevE/)

:warning: Important: All subdomains need a themes.yml file in a location set in
~~~yaml
subdomains_themes_files: "{{ lookup('fileglob', 'files/galaxy/config/themes/*.yml', wantlist=True) }}"
~~~
the name **must** match the subdomain name.  
E.g. for example.usegalaxy.eu you need  
~~~
files/galaxy/config/themes/example.yml
~~~
Otherwise the subdomains may not be considered by the role.  

:warning:**NGINX**: Make sure that Galaxy serves all subdomains (e.g. by omit `sever_name` and set `proxy_set_header Host $http_host;` under `location /`)

## The Role can do the following:
### Create static directories per subdomain when `subdomains_serve_static: true`
- Creates a full depth copy of Galaxy's static dir for each subdomain
- Copy subdomain specific static files, like `favicon.ico` or images. You need to create a directory under `subdomains_ansible_file_path` for each subdomain (and named like the subdomain) with subdirectories for each value in `subdomains_static_keys`, if you have a file in it (can be omitted otherwise). The file structure could look like the following:
~~~bash
files
└── galaxy
    ├── static
    │   └── climate
    │       ├── dist
    │       ├── images
    │       └── welcome.html
    ├── themes
    │   └── climate.yml
    ├── themes_conf.yml
~~~
### Add "static-per-host" configuration to `galaxy.yml` when `subdomains_serve_static_config: true`
- Sets `static_enabled: true` and creates entries for each value in `subdomains_static_keys` for each subdomain
### Manage themes configuration files when `subdomains_themes_per_host: true`
- Appends `themes_conf.yml` to each file in `subdomains_themes_files` and copy them over to `galaxy_config_dir`
- Adds entries for `themes_config_file_by_host` for each subdomain in `galaxy.yml`

### Template "Global Host Filters" when `subdomains_host_filters: true`
This Python script is still needed for managing tool sections for each subdomain.

## Role Variables

See defaults for all variables. Most important variables were explained in the previous section.


## Example Playbook

```
- name: usegalaxy.aq
  hosts: galaxy
  become: true
  vars:
    subdomains_themes_conf_path: files/galaxy/themes.yml
    subdomains_themes_files: "{{ lookup('fileglob', 'files/galaxy/themes/*.yml', wantlist=True) }}"
    subdomains_base_indentation: "    "
    subdomains_instance_domain: example.org
    subdomains_tool_sections:
      - name: assembly
        tool_sections:
          - "hicexplorer"
          - "graph_display_data"
          - "peak_calling"
          - "assembly"
          - "annotation"
          - "genome_diversity"
          - "multiple_alignments"
        extra_tool_labels:
          - "proteomics"

  roles:
    - galaxyproject.galaxy
    - usegalaxy_eu.galaxy_subdomains
    - galaxyproject.nginx
```

## Role Dependencies

- galaxyproject.galaxy (depends on Galaxy variables like `galaxy_server_dir`)

Many variables from that role are set. It is assumed you will use that in your playbook as well.

## License

GPLv3

## Authors
- [Helena Rasche](@hexylena)
- [Björn Grüning](@bgruening)
- [Gianmauro Cuccuru](@gmauro)
- [Simon Gladman](@slugger70)
- [Mira Kuntz](@mira-miracoli)
