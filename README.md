Role Name
=========

Role to install and configure clockwork.

Requirements
------------

This was only tested with ansible 2.12.

Role Variables
--------------

Here is a list of the important variables. Those marked with [SECRET]
contain private information that should be kept secure in a vault or
something similar.

 - `clockwork_server_name`: The fqdn for the clockwork web server.
 - `clockwork_google_client_id`: The Google client ID for OAuth.
 - `clockwork_google_client_secret`: [SECRET] The Google client secret for OAuth.
 - `clockwork_git_deploy_key`: [SECRET] SSH private key to authenticate to github for checkout.
 - `clockwork_ssh_private_key`: [SECRET] SSH private key to connect to remote clusters
 - `clockwork_ssl_cert_file`: [SECRET] SSL certificate file for the website
 - `clockwork_ssl_cert_key`: [SECRET] SSL certificate key for the website
 - `clockwork_ldap_certificate`: [SECRET] SSL certificate for the ldap server (only useful for user sync)
 - `clockwork_ldap_private_key`: [SECRET] SSL private key for the ldap server (only useful for user sync)
 - `clockwork_secret_key`: [SECRET] Flask secret key used to secure session cookies
 - `clockwork_clusters`: A list of mappings that describe the monitored clusters.
    Here is an example with a description of the fields:
    ```
      # The name is a user-friendly name to identify this entry.  Should not contain spaces
    - name: cluster

      # This is the name of a MongoDB field that will be used to store the account name for each user on this cluster.
      # Multiple cluters can have the same field to share username
      # This must avoid clashing with builtin clockwork fields.
      account_field: cluster_username

      # This is the name of a MongoDB field that will be used to store the account update key for each user on this cluster.
      # If clusters share the account_field, they should also share the update_field.
      # If clusters share the update_field, but not the account_field, only one of them will be updated.
      # If you don't want user updates, set it to false.
      # This must avoid clashing with builtin clockwork fields.
      update_field: cluster_update_key

      # The allocations to monitor, as a list.
      # Can be set to "*" to monitor all allocations
      allocations: [ alloc1, alloc2, alloc4 ]
      allocations: "*"

      # The timezone for the cluster
      # This is used in the import scripts since slurm report times in local time with no indications of the timezone.
      timezone: America/Montreal

      # Set to false to disable remote fetching of stats.
      fetch_remote: true

      # Connection information for remote fetching
      # Optional if fetch_remote is set to false
      # will run rsync {remote_user}@{remote_hostname}:{remote_path} /some/local/path
      remote_user:
      remote_hostname:
      remote_path:

      # The interval at which to fetch stats, in minutes.
      # Must be one of 1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30
      remote_interval: 10
    ```
  - `clockwork_mongodb_connection_string`: If using the included mongod role, the default is okay.  If using an extrenal mongodb server then it should be set to match.  [SECRET] if using username and password.

There are other variables to change the local username and other minor
details, refer to the defaults/main.yml file for more information.

Example Playbook
----------------

This is a sample that uses all the defined tasks:

    - hosts: clockwork
      tasks:
        - name: Setup local MongoDB
          include_role:
            name: mila.clockwork
            tasks_from: setup_mongod
        - name: Setup clockwork server
          include_role:
            name: mila.clockwork
            tasks_from: setup_server
        - name: Setup clockwork stats collection
          include_role:
            name: mila.clockwork
            tasks_from: setup_scrape
        - name: Setup user synchcronization with google ldap
          include_role:
            name: mila.clockwork
            tasks_from: setup_users


If you want to use a pre-existing instance of MongoDB, skip the
setup_mongod step and set the correct connection string in
`clockwork_mongodb_conncetion_string`.

The setup_user step has many hardcoded choices for now which might not
be useful everywhere, instead of running it as-is, you might want to
adjust it.

License
-------

MIT
