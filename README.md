# ar_os_seed
Ansible Role that wraps openshift-applier role

## Requirements
- oc command line is available

## Role Variables
The following details:
- the parameters that should be passed to the role (aka vars)
- the defaults that are held
- the secrets that should generally be sourced from an ansible vault.

### Parameters:
| Variable                     | Description                             | Default |
| --------                     | -----------                             | ------- |
| ar_os_seed_config_dest       | Path in which to generate seeding files | null    |
| ar_os_seed_namespace         | Namespace on which to operate           | null    |
| ar_os_seed_serviceaccounts   | Service accounts to create              | null    |
| ar_os_seed_openshift_content | Openshift objects to create (see below) | null    |

For ar_os_seed_serviceaccounts the structure is:
```
[
  sa_name: '<Name of service account>'
  scc: '<Security Context Constraint to apply>'
  link_secrets: [
    {
      secret_name: '<Secret Name>'
      server: '<Associated Server>'
      username: '<User associated with the secret>'
      password: '<Password for the user>'
      email: '<Email for the user (optional)>'
      token: '<Token created for the user>'
      operation: '<Type of operation ['pull' | 'push']>'
    }, ... 
  ]
]
```

For ar_os_seed_openshift_content the structure is:
```
[
  { name: "<Name of object", template: "<Path to kubernetes template>", params: {'<key>': '<value>', ...} },
  ...
]
```

### Defaults
| Variable                       | Description                                               | Default                                                                                                                            |
| --------                       | -----------                                               | -------                                                                                                                            |
| ar_os_seed_apply_assertions    | List of assertions made before calling openshift-applier  | 'ar_os_seed_openshift_login_url' is provided                                                                                       |
| ar_os_seed_content_assertions  | List of assertions made before creating the seeding files | 'ar_os_seed_openshift_content', 'ar_os_seed_namespace', 'ar_os_seed_config_dest' and 'ar_os_seed_openshift_login_url' are provided |
| ar_os_seed_registry_secrets    | List of registry secrets required                         | if defined, registry_secrets, otherwise []                                                                                         |
| ar_os_seed_openshift_login_url | Openshift login url                                       | null                                                                                                                               |


## Dependencies

- openshift-applier
- ar_os_common (login)
- ar_os_registry_secret
- ar_os_scc_binding
- ar_os_secret_link

## Example Playbook

To generate seed config and apply to cluster:
```
- hosts: localhost
  tasks:
    - name: Include Seeding
      include_role:
        name: ar_os_seed
      vars:
        ar_os_seed_namespace: "sunshine-desserts"
        ar_os_seed_config_dest:
        ar_os_seed_serviceaccounts: [
          sa_name:
          scc:
          link_secrets: [
            {
              sa_name: 'default', 
              link_secrets: [
                { secret_name: 't2-registry-credential' }
              ]
            }       
          ]
        ]
        ar_os_seed_openshift_content: [
          { name: "app-certs", template: "/templates/amq-interconnect-certs.yml" },
          { name: "interconnect-config-map", template: "/templates/amq-interconnect-qdrouter-cm.yml" },
          { name: "interconnect-sasl-config-map", template: "/templates/sasl-qdrouterd-cm.yml" },
          { name: "app-users", template: "/templates/users.yml" },
          { name: "interconnect-app", template: "/templates/interconnect-k8s-template.yml",
            params: {
              APPLICATION_NAME: "interconnect",
              QDROUTERD_CONF_NAME: "interconnect-config-map",
              SASL_CONF_NAME: "interconnect-sasl-config-map",
              USERS_SECRET_FILES_NAME: "app-users",
              CERT_SECRET_NAME: "app-certs",
              IMAGE_NAME: "amq-interconnect:latest",
              IMAGE_STREAM_NAMESPACE: "openshift",
              APP_NAME: "interconnect"
            }
          }        
        ]
```

## License

MIT / BSD

## Author Information

This role was created in 2020 by David Stewart (dstewart@redhat.com)
